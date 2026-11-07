# ENTREGABLE 5: CÓDIGO FUENTE DE LA APLICACIÓN WEB CON IA (`app.py` / `05_Aplicacion_IA.zip`)
# Lenguaje: Python 3.9+ | Framework: Streamlit | Librerías: pandas, numpy, matplotlib

import streamlit as st
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# ---------------------------------------------------------
# CONFIGURACIÓN DE PÁGINA
# ---------------------------------------------------------
st.set_page_config(
    page_title="El Tercer Cielo - Simulador Económico IA",
    page_icon="🍷",
    layout="wide"
)

# ---------------------------------------------------------
# ENCABEZADO Y DATOS DE LA EMPRESA
# ---------------------------------------------------------
st.title("🍷 Vinoteca & Cava Artesanal 'El Tercer Cielo'")
st.subheader("Plataforma de Inteligencia Económica y Simulación Financiera (Medellín, Antioquia)")
st.markdown("""
Esta herramienta interactiva evalúa la viabilidad financiera de **El Tercer Cielo** mediante la simulación de variables macroeconómicas 
(TRM e Inflación) y genera recomendaciones estratégicas respaldadas por un motor de reglas predictivas con IA.
""")
st.divider()

# ---------------------------------------------------------
# BARRA LATERAL (ENTRADAS DE DATOS)
# ---------------------------------------------------------
st.sidebar.header("⚙️ Variables Macroeconómicas y Mercado")

trm_input = st.sidebar.number_input(
    "TRM Proyectada (COP / USD):",
    min_value=3000.0,
    max_value=6000.0,
    value=4250.0,
    step=50.0
)

ipc_input = st.sidebar.slider(
    "Inflación IPC Estimada (%):",
    min_value=1.0,
    max_value=15.0,
    value=4.1,
    step=0.1
)

precio_input = st.sidebar.number_input(
    "Precio Venta por Botella (COP):",
    min_value=40000.0,
    max_value=250000.0,
    value=85000.0,
    step=5000.0
)

ventas_input = st.sidebar.slider(
    "Ventas Estimadas (Botellas/mes):",
    min_value=300,
    max_value=4000,
    value=1800,
    step=50
)

# ---------------------------------------------------------
# CÁLCULOS DEL MODELO ECONÓMICO
# ---------------------------------------------------------
trm_ref = 4000.0
var_trm = (trm_input - trm_ref) / trm_ref

cogs_importado = 30000.0 * (1 + var_trm)
cogs_nacional = 18000.0 * (1 + (ipc_input / 100))
costo_var_unitario = cogs_importado + cogs_nacional

costos_fijos_base = 27000000.0
costos_fijos_ajustados = costos_fijos_base * (1 + (ipc_input / 100) * 0.5)

ingresos_totales = precio_input * ventas_input
costos_var_totales = costo_var_unitario * ventas_input
margen_contribucion = ingresos_totales - costos_var_totales
utilidad_operativa = margen_contribucion - costos_fijos_ajustados
margen_op_pct = (utilidad_operativa / ingresos_totales) * 100 if ingresos_totales > 0 else 0

margen_unitario = precio_input - costo_var_unitario
pe_unidades = costos_fijos_ajustados / margen_unitario if margen_unitario > 0 else 0

# ---------------------------------------------------------
# DESPLIEGUE DE MÉTRICAS (OUTPUTS)
# ---------------------------------------------------------
c1, c2, c3, c4 = st.columns(4)
c1.metric("Ingresos Mensuales", f"${ingresos_totales:,.0f} COP")
c2.metric("Costo Var. Unitario", f"${costo_var_unitario:,.0f} COP", delta=f"{var_trm*100:.1f}% vs Ref")
c3.metric("Utilidad Operativa", f"${utilidad_operativa:,.0f} COP", delta=f"{margen_op_pct:.1f}% Margen")
c4.metric("Punto de Equilibrio", f"{int(pe_unidades)} Botellas/mes")

st.divider()

# ---------------------------------------------------------
# MOTOR DE IA (RECOMENDACIÓN Y DIAGNÓSTICO)
# ---------------------------------------------------------
st.header("🤖 Recomendación Inteligente - El Tercer Cielo AI")

def motor_ia_el_tercer_cielo(trm, ipc, margen, pe, ventas):
    alertas = []
    if margen < 10.0:
        estado = "CRÍTICO"
        alertas.append("⚠️ La TRM elevada está erosionando el margen operativo de las líneas importadas.")
        rec = ("ACCION INMEDIATA: Reajustar precio en 8% o incrementar la participación de la Línea Artesanal "
               "Antioqueña en un 35% para compensar los costos cambiarios.")
    elif 10.0 <= margen < 22.0:
        estado = "MODERADO"
        alertas.append("⚡ Margen en zona neutra. El volumen sostiene la operación física en El Poblado.")
        rec = ("ACCION SUGERIDA: Fomentar catas privadas corporativas e implementar maridaje cruzado "
               "para subir el ticket promedio.")
    else:
        estado = "ÓPTIMO"
        alertas.append("✅ Excelente salud financiera y alto margen operativo.")
        rec = ("ACCION SUGERIDA: Recomendar la apertura del segundo punto de venta o reinvertir en la "
               "plataforma e-commerce nacional.")
    return estado, alertas, rec

estado_ia, alertas_ia, rec_ia = motor_ia_el_tercer_cielo(trm_input, ipc_input, margen_op_pct, pe_unidades, ventas_input)

if estado_ia == "ÓPTIMO":
    st.success(f"**Estado del Negocio Evaluado por IA:** {estado_ia}")
elif estado_ia == "MODERADO":
    st.warning(f"**Estado del Negocio Evaluado por IA:** {estado_ia}")
else:
    st.error(f"**Estado del Negocio Evaluado por IA:** {estado_ia}")

for a in alertas_ia:
    st.write(a)

st.info(f"**Diagnóstico Estratégico:**\n\n{rec_ia}")

st.divider()

# ---------------------------------------------------------
# GRÁFICOS INTERACTIVOS
# ---------------------------------------------------------
st.header("📊 Análisis Gráfico")
t1, t2 = st.tabs(["Punto de Equilibrio", "Sensibilidad TRM vs Utilidad"])

with t1:
    fig, ax = plt.subplots(figsize=(8, 3.5))
    x = np.linspace(100, 3000, 50)
    y_ing = precio_input * x
    y_cost = costos_fijos_ajustados + (costo_var_unitario * x)
    ax.plot(x, y_ing, label="Ingresos Totales", color="green")
    ax.plot(x, y_cost, label="Costos Totales", color="red")
    ax.axvline(x=pe_unidades, color="blue", linestyle=":", label=f"Equilibrio ({int(pe_unidades)} bot.)")
    ax.set_title("Equilibrio Operativo (El Tercer Cielo)")
    ax.legend()
    ax.grid(True, alpha=0.3)
    st.pyplot(fig)

with t2:
    fig2, ax2 = plt.subplots(figsize=(8, 3.5))
    trm_range = np.linspace(3500, 5500, 20)
    u_range = []
    for t in trm_range:
        v_t = (t - 4000.0) / 4000.0
        c_v = (30000.0 * (1 + v_t)) + cogs_nacional
        u = (precio_input * ventas_input) - (c_v * ventas_input) - costos_fijos_ajustados
        u_range.append(u / 1e6)
    ax2.plot(trm_range, u_range, color="purple", marker="o")
    ax2.axhline(y=0, color="black", linestyle="--")
    ax2.set_title("Sensibilidad: TRM vs Utilidad Operativa (Millones COP)")
    ax2.grid(True, alpha=0.3)
    st.pyplot(fig2)

st.caption("Vinoteca y Cava Artesanal El Tercer Cielo S.A.S. - Medellín, Antioquia (2026)")