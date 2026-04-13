# Portfolio

import pandas as pd
import re
from rapidfuzz import fuzz

# =========================
# LOAD
# =========================
df_old = pd.read_excel("c:/Thesis/Ex-factory & UN sales per month_Melany.xlsx")

df_new = pd.read_excel("C:/Thesis/MKT_RETAIL 2024-2025 MTH.xlsx")  

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

    for col in df.columns:
        if "molecule old" in col:
            rename_dict[col] = "molecule_old"
        elif "molecule" in col:
            rename_dict[col] = "molecule"
        elif col == "cls" or "rimborso" in col:
            rename_dict[col] = "reimbursement_class"

    df = df.rename(columns=rename_dict)

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

    for col in ["product", "pack", "manufacturer"]:
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
    x = re.sub(r"\b(compr|cpr|compresse|riv\w*)\b", "cpr", x)
    x = re.sub(r"\b(capsule|caps)\b", "caps", x)
    x = re.sub(r"\s+", " ", x)
    return x.strip()

df_old["pack"] = df_old["pack"].apply(clean_pack)
df_new["pack"] = df_new["pack"].apply(clean_pack)

# =========================
# df_new → LONG → PIVOT MENSILE
# =========================
year_cols = [col for col in df_new.columns if re.search(r"\d{1,2}/\d{4}", str(col))]

df_long = df_new.melt(
    id_vars=["atc4","product","pack","pack_raw","manufacturer","reimbursement_class","metric"],
    value_vars=year_cols,
    var_name="date",
    value_name="value"
)

# filtri
df_long = df_long[df_long["metric"].str.lower().str.contains("sell-in")]
df_long = df_long[~df_long["metric"].str.lower().str.contains("usd")]

# metric type
df_long["metric_type"] = df_long["metric"].str.lower().apply(
    lambda x: "units" if re.search(r"\bun\b", x)
    else ("eur" if re.search(r"\beur\b", x) else None)
)

df_long = df_long[df_long["metric_type"].notna()]

# date parsing
df_long["date"] = pd.to_datetime(df_long["date"], format="%m/%Y", errors="coerce")
df_long["year_month"] = df_long["date"].dt.to_period("M")

# colonna finale
df_long["new_col"] = "sellin_" + df_long["metric_type"] + "_" + df_long["year_month"].astype(str)

# pivot
df_new_clean = df_long.pivot_table(
    index=["atc4","product","pack_raw","manufacturer","reimbursement_class"],
    columns="new_col",
    values="value",
    aggfunc="sum"
).reset_index()

# =========================
# df_old → RENAME MENSILE
# =========================
new_cols_old = {}

for col in df_old.columns:
    c = col.lower()
    date_match = re.search(r"(\d{1,2}/\d{4})", c)

    if date_match:
        date = pd.to_datetime(date_match.group(1), format="%m/%Y")
        ym = date.to_period("M")

        if "sell-in" in c and "eur" not in c:
            new_cols_old[col] = f"sellin_units_{ym}"
        elif "eur" in c:
            new_cols_old[col] = f"sellin_eur_{ym}"

df_old = df_old.rename(columns=new_cols_old)

# =========================
# CREA KEY
# =========================
def create_key(df):
    return (
        df["product"].fillna("") + "_" +
        df["pack_raw"].fillna("") + "_" +
        df["manufacturer"].fillna("") + "_" +
        df["reimbursement_class"].fillna("")
    )

df_old["key"] = create_key(df_old)
df_new_clean["key"] = create_key(df_new_clean)

# =========================
# MERGE
# =========================
df2 = df_old.merge(
    df_new_clean,
    on="key",
    how="outer",
    suffixes=("_old","_new"),
    indicator=True
)

# =========================
# RICOSTRUISCI COLONNE BASE
# =========================
df2["atc4"] = df2.get("atc4_old").combine_first(df2.get("atc4_new"))
df2["product"] = df2.get("product_old").combine_first(df2.get("product_new"))
df2["pack_raw"] = df2.get("pack_raw_old").combine_first(df2.get("pack_raw_new"))
df2["manufacturer"] = df2.get("manufacturer_old").combine_first(df2.get("manufacturer_new"))
df2["reimbursement_class"] = df2.get("reimbursement_class_old").combine_first(df2.get("reimbursement_class_new"))

# =========================
# CONTROLLO COMPLETEZZA
# =========================
key_cols = ["product","pack_raw","manufacturer","reimbursement_class"]

n_old = df_old[key_cols].drop_duplicates().shape[0]
n_new = df_new_clean[key_cols].drop_duplicates().shape[0]
n_final = df2[key_cols].drop_duplicates().shape[0]

print("\n=== COUNT PRODOTTI ===")
print("old:", n_old)
print("new:", n_new)
print("final:", n_final)

# =========================
# OUTPUT
# =========================
base_cols = ["atc4","product","pack_raw","manufacturer","reimbursement_class"]
sell_cols = [c for c in df2.columns if c.startswith("sellin_")]

df_final = df2[base_cols + sell_cols].copy()

# duplicati
print("\nDuplicati:",
df_final.duplicated(key_cols).sum())

# =========================
# SAVE
# =========================
df_final.to_excel("c:/Thesis/final_dataset_monthly_2.xlsx", index=False)

print("\n✅ FILE FINALE CREATO!")

# =========================
# CONTROLLO COMPLETEZZA PRODOTTI
# =========================

key_cols = ["product","pack_raw","manufacturer","reimbursement_class"]

# -------------------------
# 1. COUNT BASE
# -------------------------
n_old = df_old[key_cols].drop_duplicates().shape[0]
n_new = df_new_clean[key_cols].drop_duplicates().shape[0]
n_final = df2[key_cols].drop_duplicates().shape[0]

print("\n=== COUNT PRODOTTI UNIVOCI ===")
print(f"df_old:   {n_old}")
print(f"df_new:   {n_new}")
print(f"df_final: {n_final}")

# -------------------------
# 2. SET DI CHIAVI
# -------------------------
old_keys = set(df_old[key_cols].drop_duplicates().apply(tuple, axis=1))
new_keys = set(df_new_clean[key_cols].drop_duplicates().apply(tuple, axis=1))
final_keys = set(df2[key_cols].drop_duplicates().apply(tuple, axis=1))

# -------------------------
# 3. OVERLAP
# -------------------------
both_keys = old_keys & new_keys
only_old_keys = old_keys - new_keys
only_new_keys = new_keys - old_keys

print("\n=== OVERLAP ===")
print(f"Comuni:     {len(both_keys)}")
print(f"Solo OLD:   {len(only_old_keys)}")
print(f"Solo NEW:   {len(only_new_keys)}")

# -------------------------
# 4. ATTESO VS OTTENUTO
# -------------------------
expected_total = len(old_keys | new_keys)

print("\n=== CONTROLLO FINALE ===")
print(f"Attesi (union): {expected_total}")
print(f"Finale:         {n_final}")

if n_final == expected_total:
    print("✅ DATASET COMPLETO (nessuna perdita)")
else:
    print("❌ ATTENZIONE: mancano prodotti")

# -------------------------
# 5. IDENTIFICA PRODOTTI PERSI
# -------------------------
missing_products = (old_keys | new_keys) - final_keys

print("\nProdotti mancanti:", len(missing_products))

# opzionale: salva lista
missing_df = pd.DataFrame(list(missing_products), columns=key_cols)
missing_df.to_excel("c:/Thesis/prodotti_mancanti.xlsx", index=False)

# -------------------------
# 6. DUPLICATI
# -------------------------
duplicates = df2.duplicated(key_cols).sum()

print("\nDuplicati:", duplicates)
