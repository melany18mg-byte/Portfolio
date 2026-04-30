import pandas as pd
import numpy as np
import re

# =========================
# 1. LOAD
# =========================
df_old = pd.read_excel("c:/Thesis/Retail FULL MAT 2016_2023 DWN.xlsx")
df_new = pd.read_excel("c:/Thesis/Retail 2023-25.xlsx")

# =========================
# CLEAN BASE
# =========================
def clean_df(df):

    df.columns = (
        df.columns
        .str.replace("\n", " ", regex=False)
        .str.strip()
        .str.lower()
    )

    rename_dict = {}
    cols_to_drop = []

    for col in df.columns:
        if "single/combined molecule old" in col:
            cols_to_drop.append(col)
        elif "single" in col and "molecule" in col and "old" not in col:
            rename_dict[col] = "single_combined_molecule"
        elif col == "cls" or "rimborso" in col:
            rename_dict[col] = "reimbursement_class"

    df = df.rename(columns=rename_dict)
    df = df.drop(columns=cols_to_drop, errors="ignore")

    if "reimbursement_class" not in df.columns:
        df["reimbursement_class"] = None

    df["reimbursement_class"] = (
        df["reimbursement_class"]
        .astype(str)
        .str.replace("*", "", regex=False)
        .str.strip()
        .str.lower()
    )

    mapping = {
        "farmaci automedic": "classe c",
        "sop": "classe c",
        "otc": "classe c",
        "otc s/registraz": "classe c",
        "farmaci sop": "classe c",
        "farmaci s.p.": "classe c",
        "limt px reg ussl-a": "classe a",
        "a": "classe a",
        "c": "classe c"
    }   
    

    df["reimbursement_class"] = df["reimbursement_class"].replace(mapping)

    def clean_text(x):
        if pd.isna(x):
            return None
        return str(x).lower().strip()

    for col in ["product", "pack", "manufacturer", "single_combined_molecule"]:
        if col in df.columns:
            df[col] = df[col].apply(clean_text)

    return df

df_old = clean_df(df_old)
df_new = clean_df(df_new)

# =========================
# PACK RAW
# =========================
df_old["pack_raw"] = df_old["pack"]
df_new["pack_raw"] = df_new["pack"]

# =========================
# CLEAN MANUFACTURER
# =========================
def clean_manufacturer(x):
    if pd.isna(x):
        return None
    x = str(x).lower().strip()
    x = re.sub(r"\b(spa|srl|ltd|gmbh|inc|sa)\b", "", x)
    x = re.sub(r"\s+", " ", x)
    return x.strip()

df_old["manufacturer"] = df_old["manufacturer"].apply(clean_manufacturer)
df_new["manufacturer"] = df_new["manufacturer"].apply(clean_manufacturer)

# =========================
# CLEAN PACK
# =========================
def clean_pack(x):
    if pd.isna(x):
        return None
    x = str(x).lower()
    x = re.sub(r"[.,]", "", x)
    x = re.sub(r"\b(compr|cpr|compresse)\b", "cpr", x)
    x = re.sub(r"\b(capsule|caps)\b", "caps", x)
    x = re.sub(r"[x×]", "x", x)
    x = re.sub(r"\s+", " ", x)
    return x.strip()

df_old["pack"] = df_old["pack"].apply(clean_pack)
df_new["pack"] = df_new["pack"].apply(clean_pack)

# =========================
# df_new → MAT LONG
# =========================
mat_cols = [col for col in df_new.columns if "mat" in col.lower()]

df_long = df_new.melt(
    id_vars=[
        "atc4","product","pack","pack_raw","manufacturer",
        "reimbursement_class","metric",
        "single_combined_molecule","protection"
    ],
    value_vars=mat_cols,
    var_name="mat_date",
    value_name="value"
)

# filtri
df_long = df_long[df_long["metric"].str.contains("sell", case=False, na=False)]
df_long = df_long[~df_long["metric"].str.contains("usd", case=False, na=False)]

# units / eur
df_long["metric_type"] = df_long["metric"].str.lower().apply(
    lambda x: "units" if "un" in x else ("eur" if "eur" in x else None)
)
df_long = df_long[df_long["metric_type"].notna()]

# parse MAT Dec YYYY
def parse_mat_date(x):
    match = re.search(r"(dec)\s*(\d{4})", str(x).lower())
    if match:
        return f"{match.group(2)}-12"
    return None

df_long["ym"] = df_long["mat_date"].apply(parse_mat_date)

# colonne finali
df_long["col_name"] = "sellin_" + df_long["metric_type"] + "_" + df_long["ym"]

# pivot
df_new_mat = df_long.pivot_table(
    index=[
        "atc4","product","pack_raw","manufacturer",
        "reimbursement_class","single_combined_molecule"
    ],
    columns="col_name",
    values="value",
    aggfunc="sum"
).reset_index()

# =========================
# df_old → MAT rename
# =========================
rename_old = {}

for col in df_old.columns:
    m = re.search(r"(\d{1,2}/\d{4})", col)
    if m:
        date = pd.to_datetime(m.group(1), format="%m/%Y")
        ym = date.to_period("M")
        if "eur" in col:
            rename_old[col] = f"sellin_eur_{ym}"
        else:
            rename_old[col] = f"sellin_units_{ym}"

df_old = df_old.rename(columns=rename_old)

# =========================
# KEY
# =========================
def create_key(df):
    return (
        df["product"].fillna("") + "_" +
        df["pack_raw"].fillna("") + "_" +
        df["manufacturer"].fillna("") + "_" +
        df["reimbursement_class"].fillna("") + "_" +
        df["single_combined_molecule"].fillna("")
    )

df_old["key"] = create_key(df_old)
df_new_mat["key"] = create_key(df_new_mat)

# =========================
# MERGE
# =========================
df = df_old.merge(df_new_mat, on="key", how="outer", suffixes=("_old","_new"))

# =========================
# REBUILD COLS
# =========================
for col in ["atc4","product","pack_raw","manufacturer","reimbursement_class","single_combined_molecule"]:
    df[col] = df.get(col+"_old").combine_first(df.get(col+"_new"))

# =========================
# DROP OLD/NEW DUPLICATES
# =========================
df = df.drop(columns=[c for c in df.columns if c.endswith("_old") or c.endswith("_new")])

# =========================
# ORDER COLUMNS
# =========================
time_cols = sorted([c for c in df.columns if c.startswith("sellin_")])

base_cols = [
    "atc4","product","pack_raw","manufacturer",
    "reimbursement_class","single_combined_molecule"
]

other_cols = [
    c for c in df.columns
    if c not in base_cols and c not in time_cols
]

df_final = df[base_cols + other_cols + time_cols]

# =========================
# CHECK
# =========================
key_cols = ["product","pack_raw","manufacturer","reimbursement_class","single_combined_molecule"]

print("\n=== CONTROLLO ===")
print(f"Duplicati: {df_final.duplicated(subset=key_cols).sum()}")

# =========================
# SAVE
# =========================
df_final.to_csv("c:/Thesis/dataset_final_clean.csv", index=False)



# =========================
# LOAD FILE ESTERNI
# =========================
df_final = pd.read_excel("c:/Thesis/2016_2025 MAT.xlsx")
df_dash  = pd.read_excel("c:/Thesis/dashboard_2.xlsx")


# =========================
# CLEAN COLONNE
# =========================
df_dash.columns = (
    df_dash.columns
    .str.lower()
    .str.strip()
    .str.replace(r"[^a-z0-9]", "_", regex=True)
)


# =========================
# NORMALIZZAZIONE PRODUCT
# =========================
def normalize_product(x):
    if pd.isna(x):
        return ""
    x = str(x).lower().strip()
    x = re.sub(r"[^a-z0-9 ]", " ", x)
    x = re.sub(r"\s+", " ", x)
    return x.strip()

df_final["key"] = df_final["product"].apply(normalize_product)
df_dash["key"]  = df_dash["product"].apply(normalize_product)

# =========================
#  FIX DUPLICATI (FONDAMENTALE)
# =========================
df_dash = (
    df_dash
    .sort_values(by=["competitor"], na_position="last")
    .groupby("key", as_index=False)
    .first()
)


# =========================
# MERGE 1 → DASHBOARD
# =========================
df_merged = df_final.merge(
    df_dash[
        [
            "sing_comb_act_ing_s_",
            "gx_market__iqvia_",
            "gx_market__sdz_",
            "molecule_adj__name",
            "molecule_list_name"
        ]
    ],
    on="key",
    how="left"
)


# =========================
# CHECK MERGE
# =========================
print("\n=== CHECK MERGE ===")
print("Righe finali:", len(df_merged))
print("Prodotti unici:", df_merged["key"].nunique())

print("Missing competitor:", df_merged["competitor"].isna().sum())
print("Missing business_rule:", df_merged["business_rule"].isna().sum())

# =========================
# SAVE (VELOCE E SICURO)
# =========================
print("\nInizio salvataggio.")

df_merged.to_csv("c:/Thesis/2016_2025.csv", index=False)

print(" FILE SALVATO ")
