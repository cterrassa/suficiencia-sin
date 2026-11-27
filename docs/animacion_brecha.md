# La brecha del caso base, mes a mes

Complemento del tutorial «Suficiencia energética del SIN 2026-2030 y expansión de mínimo costo» (`Tutorial_Suficiencia_SIN.ipynb`). Esta animación estaba en la sección 4.2. La pasamos al repositorio porque no hace falta para responder la pregunta: muestra de otra forma la brecha que ya da la figura de esa sección.

**Para verla:** https://cterrassa.github.io/suficiencia-sin/docs/animacion_brecha.html

## Qué muestra

La figura de la sección 4.2 del tutorial suma cada período en una barra. Esta animación muestra cómo se forma esa suma: la demanda media de cada mes, por día, frente a la oferta firme diaria del caso base (sección 3.3), de diciembre de 2026 a noviembre de 2030.

El texto lleva la cuenta de la brecha acumulada del período, que vuelve a cero cada diciembre. Se reproduce con el botón ▶ o se recorre con el control deslizante.

## Lectura

La brecha no viene de unos pocos meses de demanda alta: se acumula a lo largo del año. En 2026-2027 la oferta física del caso base es de 251,3 GWh por día, y la demanda media la supera en 11 de los 12 meses (solo enero, el mes de menor demanda, queda por debajo); en 2027-2028 pasa lo mismo, y desde 2028-2029 la supera en todos. En 2029-2030 la oferta sube a 263,5 GWh por día con la entrada de los proyectos con OEF, pero la demanda sigue por encima todo el año. Cada noviembre la cuenta cierra exactamente en la brecha de la figura de la sección 4.2 (1.770, 2.809, 5.168 y 4.585 GWh), porque las dos vistas salen de los mismos datos.

## Cómo reproducirla

Con el tutorial ejecutado hasta la sección 4.2, se pegan estas dos celdas después de la figura de esa sección. Usan lo que el tutorial ya definió: `demanda_upme`, `DEFINICION`, `PERIODOS`, `periodo_cxc()`, `balance()`, `es()` y `LEYENDA`.

```python
# Los 48 meses del horizonte: demanda media y oferta firme del caso base, por día
meses = demanda_upme.query("definicion == @DEFINICION and escenario == 'medio'").copy()
meses["periodo"], meses["dias"] = periodo_cxc(meses["mes"]), meses["mes"].dt.days_in_month
meses = meses[meses["periodo"].isin(PERIODOS)].sort_values("mes").reset_index(drop=True)
firme = balance("base").set_index("periodo")
meses["demanda_dia"] = meses["energia_gwh"] / meses["dias"]
meses["firme_dia"] = meses["periodo"].map(firme["oferta_firme_gwh"] / firme["dias"])
meses["acumulado_gwh"] = ((meses["demanda_dia"] - meses["firme_dia"]) * meses["dias"]).groupby(meses["periodo"]).cumsum()
cierre = meses.groupby("periodo")["acumulado_gwh"].last()
assert (cierre - firme["brecha_gwh"]).abs().max() < 0.5, "la suma de los meses no da la brecha del período"
print("Brecha acumulada a noviembre, igual a la de la sección 3.3 (GWh):", ", ".join(f"{p}: {es(v)}" for p, v in cierre.items()))

cuadros = pd.concat([meses.iloc[:i + 1].assign(cuadro=f"{m:%Y-%m}") for i, m in enumerate(meses["mes"])])
largo = cuadros.melt(id_vars=["cuadro", "mes"], value_vars=["demanda_dia", "firme_dia"], var_name="serie", value_name="gwh_dia")
SERIES = {"demanda_dia": "Demanda media de la UPME", "firme_dia": "Oferta firme del caso base"}
largo["serie"] = largo["serie"].map(SERIES)


def rotulo(fila):
    """Texto de un cuadro: brecha acumulada del período hasta ese mes."""
    palabra = "faltan" if fila.acumulado_gwh > 0 else "sobran"
    gwh = f"{abs(fila.acumulado_gwh):,.0f}".replace(",", ".")
    return f"Período {fila.periodo}, hasta {fila.mes:%Y-%m}: {palabra} {gwh} GWh firmes"
```

```python
rango_y = [meses[["demanda_dia", "firme_dia"]].min().min() * 0.97, meses[["demanda_dia", "firme_dia"]].max().max() * 1.04]
fig = px.line(largo, x="mes", y="gwh_dia", color="serie", animation_frame="cuadro", markers=True,
              color_discrete_map={SERIES["demanda_dia"]: "#2a78d6", SERIES["firme_dia"]: "#1f1f1f"},
              range_x=[meses["mes"].min() - pd.Timedelta(days=20), meses["mes"].max() + pd.Timedelta(days=20)],
              range_y=rango_y, labels={"mes": "Mes", "gwh_dia": "GWh por día", "serie": "Serie", "cuadro": "Mes"},
              title="La brecha del caso base, mes a mes: demanda media frente a oferta firme (GWh por día)")
texto = dict(mode="text", x=[meses["mes"].min()], y=[rango_y[1] * 0.995], textposition="bottom right",
             showlegend=False, hoverinfo="skip", textfont=dict(size=14, color="#1f1f1f"))
fig.add_trace(go.Scatter(text=[rotulo(meses.iloc[0])], **texto))
for cuadro, (_, fila) in zip(fig.frames, meses.iterrows()):
    cuadro.data += (go.Scatter(text=[rotulo(fila)], **texto),)
fig.update_traces(hovertemplate="%{x|%Y-%m}: %{y:,.1f} GWh por día<extra></extra>", selector=dict(mode="lines+markers"))
for cuadro in fig.frames:
    for traza in cuadro.data[:2]:
        traza.hovertemplate = "%{x|%Y-%m}: %{y:,.1f} GWh por día<extra></extra>"
fig.layout.updatemenus[0].buttons[0].args[1]["frame"]["duration"] = 150
fig.layout.updatemenus[0].buttons[0].args[1]["transition"]["duration"] = 0
fig.layout.sliders[0].currentvalue.prefix = "Mes: "
fig.update_layout(height=540, legend=dict(LEYENDA, y=1.0, yanchor="bottom"), margin=dict(t=110), xaxis_title="Mes",
                  xaxis_tickformat="%Y-%m", separators=",.")       # leyenda arriba: abajo van los controles
fig.show()
```

## Funciones que usa

| Función | Qué hace | Parámetros que usamos | Por qué así |
|---|---|---|---|
| `px.line(..., animation_frame=)` | Dibuja una figura por cada valor de la columna y las une en una animación, con botón y control deslizante | `animation_frame="cuadro"`; `range_x` y `range_y` fijos | Cada cuadro trae los meses hasta ese mes, así que la línea crece; con los ejes fijos, la escala no salta de un cuadro a otro |
| `go.Scatter(mode="text")` | Una traza que solo muestra texto | Una traza por cuadro, añadida a `fig.frames` | El texto cambia con cada cuadro, igual que las líneas |
| `layout.updatemenus` | Los botones de la animación | La duración de cada cuadro | Un mes cada 0,15 segundos: los 48 meses en unos 7 segundos |
