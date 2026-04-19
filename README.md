import pandas as pd
import re
from rapidfuzz import fuzz

# =========================
# LOAD
# =========================
df_mkt = pd.read_excel("c:/Thesis/DATASET_FINALE.xlsx")
import pandas as pd
import re

# =====================================
# 1. CREA IL MERCATO
# mercato = molecule_adj_name + frm2
# =====================================
def clean_market_text(x):
    if pd.isna(x):
        return ""
    x = str(x).strip().lower()
    x = re.sub(r"\s+", " ", x)
    return x

df_mkt["molecule_adj_clean"] = df_mkt["molecule_adj_name"].apply(clean_market_text)
df_mkt["frm2_clean"] = df_mkt["frm2"].apply(clean_market_text)

df_mkt["market_id"] = (
    df_mkt["molecule_adj_clean"] + " | " + df_mkt["frm2_clean"]
)

df_mkt["market_name"] = (
    df_mkt["molecule_adj_name"].astype(str).str.strip() + " | " +
    df_mkt["frm2"].astype(str).str.strip()
)

# =====================================
# 2. IDENTIFICA COLONNE MENSILI
# =====================================
eur_cols = [
    c for c in df_mkt.columns
    if c.startswith("sellin_eur_") and re.fullmatch(r"sellin_eur_\d{4}-\d{2}", c)
]

units_cols = [
    c for c in df_mkt.columns
    if c.startswith("sellin_units_") and re.fullmatch(r"sellin_units_\d{4}-\d{2}", c)
]

company_col = "manufacturer"
product_col = "Product"

# =====================================
# 3. MARKET SHARE MENSILE
# - prodotto
# - azienda
# =====================================
for col in eur_cols:
    suffix = col.replace("sellin_eur_", "")

    market_total_col = f"market_total_eur_{suffix}"
    product_share_col = f"product_market_share_eur_{suffix}"
    company_sales_col = f"company_sales_eur_{suffix}"
    company_share_col = f"company_market_share_eur_{suffix}"

    # totale mercato
    df_mkt[market_total_col] = df_mkt.groupby("market_id")[col].transform("sum")

    # share prodotto
    df_mkt[product_share_col] = (
        df_mkt[col] / df_mkt[market_total_col]
    ).replace([float("inf"), -float("inf")], pd.NA)

    # vendite azienda nel mercato
    df_mkt[company_sales_col] = df_mkt.groupby(["market_id", company_col])[col].transform("sum")

    # share azienda
    df_mkt[company_share_col] = (
        df_mkt[company_sales_col] / df_mkt[market_total_col]
    ).replace([float("inf"), -float("inf")], pd.NA)

for col in units_cols:
    suffix = col.replace("sellin_units_", "")

    market_total_col = f"market_total_units_{suffix}"
    product_share_col = f"product_market_share_units_{suffix}"
    company_sales_col = f"company_sales_units_{suffix}"
    company_share_col = f"company_market_share_units_{suffix}"

    # totale mercato
    df_mkt[market_total_col] = df_mkt.groupby("market_id")[col].transform("sum")

    # share prodotto
    df_mkt[product_share_col] = (
        df_mkt[col] / df_mkt[market_total_col]
    ).replace([float("inf"), -float("inf")], pd.NA)

    # vendite azienda nel mercato
    df_mkt[company_sales_col] = df_mkt.groupby(["market_id", company_col])[col].transform("sum")

    # share azienda
    df_mkt[company_share_col] = (
        df_mkt[company_sales_col] / df_mkt[market_total_col]
    ).replace([float("inf"), -float("inf")], pd.NA)

# converte in percentuale
monthly_share_cols = [
    c for c in df_mkt.columns
    if re.fullmatch(r"(product|company)_market_share_(eur|units)_\d{4}-\d{2}", c)
]

for c in monthly_share_cols:
    df_mkt[c] = (df_mkt[c] * 100).round(2)

# =====================================
# 4. CREA TOTALI ANNUALI PER RIGA
# =====================================
eur_by_year = {}
for c in eur_cols:
    year = c.replace("sellin_eur_", "")[:4]
    eur_by_year.setdefault(year, []).append(c)

units_by_year = {}
for c in units_cols:
    year = c.replace("sellin_units_", "")[:4]
    units_by_year.setdefault(year, []).append(c)

for year, cols in eur_by_year.items():
    df_mkt[f"sellin_eur_{year}"] = df_mkt[cols].sum(axis=1)

for year, cols in units_by_year.items():
    df_mkt[f"sellin_units_{year}"] = df_mkt[cols].sum(axis=1)

# =====================================
# 5. MARKET SHARE ANNUALE
# - prodotto
# - azienda
# =====================================
annual_eur_cols = [
    c for c in df_mkt.columns
    if re.fullmatch(r"sellin_eur_\d{4}", c)
]

annual_units_cols = [
    c for c in df_mkt.columns
    if re.fullmatch(r"sellin_units_\d{4}", c)
]

for col in annual_eur_cols:
    suffix = col.replace("sellin_eur_", "")

    market_total_col = f"market_total_eur_{suffix}"
    product_share_col = f"product_market_share_eur_{suffix}"
    company_sales_col = f"company_sales_eur_{suffix}"
    company_share_col = f"company_market_share_eur_{suffix}"

    # totale mercato
    df_mkt[market_total_col] = df_mkt.groupby("market_id")[col].transform("sum")

    # share prodotto
    df_mkt[product_share_col] = (
        df_mkt[col] / df_mkt[market_total_col]
    ).replace([float("inf"), -float("inf")], pd.NA)

    # vendite azienda nel mercato
    df_mkt[company_sales_col] = df_mkt.groupby(["market_id", company_col])[col].transform("sum")

    # share azienda
    df_mkt[company_share_col] = (
        df_mkt[company_sales_col] / df_mkt[market_total_col]
    ).replace([float("inf"), -float("inf")], pd.NA)

for col in annual_units_cols:
    suffix = col.replace("sellin_units_", "")

    market_total_col = f"market_total_units_{suffix}"
    product_share_col = f"product_market_share_units_{suffix}"
    company_sales_col = f"company_sales_units_{suffix}"
    company_share_col = f"company_market_share_units_{suffix}"

    # totale mercato
    df_mkt[market_total_col] = df_mkt.groupby("market_id")[col].transform("sum")

    # share prodotto
    df_mkt[product_share_col] = (
        df_mkt[col] / df_mkt[market_total_col]
    ).replace([float("inf"), -float("inf")], pd.NA)

    # vendite azienda nel mercato
    df_mkt[company_sales_col] = df_mkt.groupby(["market_id", company_col])[col].transform("sum")

    # share azienda
    df_mkt[company_share_col] = (
        df_mkt[company_sales_col] / df_mkt[market_total_col]
    ).replace([float("inf"), -float("inf")], pd.NA)

# converte in percentuale
annual_share_cols = [
    c for c in df_mkt.columns
    if re.fullmatch(r"(product|company)_market_share_(eur|units)_\d{4}", c)
]

for c in annual_share_cols:
    df_mkt[c] = (df_mkt[c] * 100).round(2)

# =====================================
# 6. INFORMAZIONI UTILI SUL MERCATO
# =====================================
df_mkt["n_products_in_market"] = df_mkt.groupby("market_id")[product_col].transform("nunique")
df_mkt["n_companies_in_market"] = df_mkt.groupby("market_id")[company_col].transform("nunique")

# ranking prodotto nel mercato sull'ultimo mese disponibile in EUR
if eur_cols:
    latest_eur_col = sorted(eur_cols)[-1]
    latest_suffix = latest_eur_col.replace("sellin_eur_", "")

    df_mkt[f"product_rank_in_market_eur_{latest_suffix}"] = (
        df_mkt.groupby("market_id")[latest_eur_col]
        .rank(method="dense", ascending=False)
    )

# ranking azienda nel mercato sull'ultimo mese disponibile in EUR
if eur_cols:
    latest_eur_col = sorted(eur_cols)[-1]
    latest_suffix = latest_eur_col.replace("sellin_eur_", "")

    tmp_company_rank = (
        df_mkt.groupby(["market_id", company_col])[latest_eur_col]
        .sum()
        .reset_index()
    )

    tmp_company_rank[f"company_rank_in_market_eur_{latest_suffix}"] = (
        tmp_company_rank.groupby("market_id")[latest_eur_col]
        .rank(method="dense", ascending=False)
    )

    df_mkt = df_mkt.merge(
        tmp_company_rank[
            ["market_id", company_col, f"company_rank_in_market_eur_{latest_suffix}"]
        ],
        on=["market_id", company_col],
        how="left"
    )

# =====================================
# 7. ORDINE COLONNE PIÙ LEGGIBILE
# =====================================
priority_cols = [
    "market_id",
    "market_name",
    "n_products_in_market",
    "n_companies_in_market"
]

remaining_cols = [c for c in df_mkt.columns if c not in priority_cols]

if "molecule_adj_name" in remaining_cols:
    pos = remaining_cols.index("molecule_adj_name") + 1
    final_cols = remaining_cols[:pos] + priority_cols + remaining_cols[pos:]
else:
    final_cols = priority_cols + remaining_cols

df_mkt = df_mkt[final_cols]

# =====================================
# 8. CHECK RAPIDI
# =====================================
print("Numero mercati:", df_mkt["market_id"].nunique())


# =====================================
# CREA DATASET LIGHT
# =====================================
base_cols = [
    "atc4",
    "molecule_adj_name",
    "top_20_rank",
    "Product",
    "pack_raw",
    "manufacturer",
    "reimbursement_class",
    "product_launch_date",
    "pack_launch_date",
    "frm2",
    "business_rule"
]

base_cols = [c for c in base_cols if c in df_mkt.columns]

share_cols = [c for c in df_mkt.columns if "market_share" in c]

df_light = df_mkt[base_cols + share_cols].copy()

df_light = df_light.sort_values(
    [c for c in ["molecule_adj_name", "frm2", "manufacturer", "Product"] if c in df_light.columns]
)


from openpyxl import load_workbook

output_file = "c:/Thesis/final_dataset_markets.xlsx"

# scrittura
with pd.ExcelWriter(output_file, engine="openpyxl") as writer:
    df_mkt.to_excel(writer, sheet_name="FULL", index=False)
    df_light.to_excel(writer, sheet_name="LIGHT", index=False)

# formatting LIGHT
wb = load_workbook(output_file)
ws = wb["LIGHT"]

# freeze (più pulito per analisi)
ws.freeze_panes = "D2"

# filtro
ws.auto_filter.ref = ws.dimensions

# larghezza colonne
for col in ws.columns:
    col_letter = col[0].column_letter
    max_length = max(
        [len(str(cell.value)) for cell in col if cell.value is not None] or [0]
    )
    ws.column_dimensions[col_letter].width = min(max_length + 2, 35)

wb.save(output_file)

print("✅ FILE CON DUE SHEET CREATO!")
