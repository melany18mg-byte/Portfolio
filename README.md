# =========================
# NORMALIZZAZIONE
# =========================
def normalize(x):
    if pd.isna(x):
        return ""
    x = str(x).lower().strip()
    x = re.sub(r"[^a-z0-9 ]", " ", x)
    x = re.sub(r"\s+", " ", x)
    return x.strip()

# =========================
# KEY DASH (più semplice)
# =========================
df_final["key_dash"] = (
    df_final["product"].apply(normalize) + "_" +
    df_final["single_combined_molecule"].apply(normalize)
)

# ⚠️ dashboard potrebbe NON avere molecule → fallback
if "single_combined_molecule" in df_dash.columns:
    df_dash["key_dash"] = (
        df_dash["product"].apply(normalize) + "_" +
        df_dash["single_combined_molecule"].apply(normalize)
    )
else:
    df_dash["key_dash"] = df_dash["product"].apply(normalize)

# =========================
# FIX DUPLICATI DASH
# =========================
df_dash = (
    df_dash
    .groupby("key_dash", as_index=False)
    .first()
)

# =========================
# MERGE CORRETTO
# =========================
df_merged = df_final.merge(
    df_dash[
        [
            "key_dash",
            "sing_comb_act_ing_s_",
            "gx_market__iqvia_",
            "gx_market__sdz_",
            "molecule_adj__name",
            "molecule_list_name"
        ]
    ],
    on="key_dash",
    how="left"
)

# =========================
# DEBUG (IMPORTANTISSIMO)
# =========================
print("\n=== DEBUG MERGE ===")
print("Match riusciti:", df_merged["gx_market__iqvia_"].notna().sum())
print("Totale righe:", len(df_merged))

missing = df_merged[df_merged["gx_market__iqvia_"].isna()]
print("Non matchati:", len(missing))
print(missing["product"].head())
