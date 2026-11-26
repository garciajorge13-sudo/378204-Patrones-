# Analisis de farmacias en Baja California - Inteligencia de Negocios
# Jorge Emilio Garcia Raygoza - 378204
# Datos: DENUE Baja California 11/2026 (INEGI), archivo denue_inegi_02_.csv
# Uso: python analisis.py denue_inegi_02_.csv

import sys
import pandas as pd

df = pd.read_csv(sys.argv[1], encoding="latin-1", dtype=str, low_memory=False)
df.columns = [c.lower() for c in df.columns]
df["municipio"] = df["municipio"].str.strip()   # San Felipe venia con espacios al final

print("Total de negocios en B.C.:", len(df))

# Oferta: farmacias (SCIAN 464111 y 464112)
far = df[df["codigo_act"].isin(["464111", "464112"])].copy()
# Demanda: consultorios medicos (6211) y hospitales (622)
sal = df[df["codigo_act"].str.startswith("6211") | df["codigo_act"].str.startswith("622")]
print("Farmacias:", len(far), "| Consultorios/hospitales:", len(sal))

# --- Calidad de datos ---
dup = far.groupby(["nom_estab", "latitud", "longitud"]).size()
print("\nFarmacias duplicadas (mismo nombre y coordenada):")
print(dup[dup > 1])
far = far.drop_duplicates(subset=["nom_estab", "latitud", "longitud"])

print("\nNegocios por anio de alta (se ven los saltos de los censos):")
print(df["fecha_alta"].str[:4].value_counts().sort_index())

# --- Hallazgo 1 y 5: consultorios por farmacia y negocios por farmacia ---
t = pd.DataFrame({
    "farmacias": far["municipio"].value_counts(),
    "salud": sal["municipio"].value_counts(),
    "negocios": df["municipio"].value_counts(),
}).fillna(0)
t["consultorios_por_farmacia"] = (t["salud"] / t["farmacias"]).round(2)
t["negocios_por_farmacia"] = (t["negocios"] / t["farmacias"]).round(0)
print("\nRazon por municipio:")
print(t.sort_values("consultorios_por_farmacia", ascending=False))
print("Promedio estatal:", round(len(sal) / len(df[df["codigo_act"].isin(["464111", "464112"])]), 2))

# --- Hallazgo 2: lo mismo por codigo postal en Tijuana y Ensenada ---
for m in ["Tijuana", "Ensenada"]:
    z = pd.DataFrame({
        "farmacias": far[far["municipio"] == m]["cod_postal"].value_counts(),
        "salud": sal[sal["municipio"] == m]["cod_postal"].value_counts(),
    }).fillna(0)
    z = z[(z["salud"] >= 15) & (z["farmacias"] > 0)]
    z["razon"] = (z["salud"] / z["farmacias"]).round(2)
    print(f"\nZonas de {m} (15+ consultorios):")
    print(z.sort_values("razon", ascending=False).head(5))

# --- Hallazgo 3: porcentaje de cadenas nacionales ---
cadenas = ["GUADALAJARA", "DEL AHORRO", "SIMILARES", "BENAVIDES", "SAN PABLO", "YZA"]
far["cadena"] = far["nom_estab"].str.upper().str.contains("|".join(cadenas), na=False)
c = far.groupby("municipio")["cadena"].agg(["sum", "count"])
c["pct_cadenas"] = (100 * c["sum"] / c["count"]).round(1)
print("\nCadenas nacionales por municipio:")
print(c.sort_values("pct_cadenas"))

# --- Hallazgo 4: farmacias registradas en 2025 y 2026 ---
far["anio"] = far["fecha_alta"].str[:4]
n = far[far["anio"].isin(["2025", "2026"])]["municipio"].value_counts()
r = pd.DataFrame({"total": far["municipio"].value_counts(), "nuevas_2025_2026": n}).fillna(0)
r["pct"] = (100 * r["nuevas_2025_2026"] / r["total"]).round(1)
print("\nFarmacias nuevas 2025-2026:")
print(r.sort_values("pct", ascending=False))
