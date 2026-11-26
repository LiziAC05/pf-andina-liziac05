# Feedback · Proyecto Final Andina Retail

**Estudiante:** Lizbeth Aquisse Castillo
**Usuario de GitHub:** `LiziAC05`
**Repo:** LiziAC05/pf-andina-liziac05
**Extensión:** E2
**Nota final:** **86 / 100**

| Criterio | Puntaje | Máximo |
|---|---|---|
| Repo y entrega | 8 | 10 |
| Ingesta con ADF | 12 | 15 |
| Orquestación observable | 11 | 15 |
| Bronce, plata, oro (UC) | 15 | 15 |
| Números de control | 15 | 15 |
| Seguridad y secretos | 5 | 10 |
| Extensión propia | 15 | 15 |
| Decisiones y defensa | 5 | 5 |
| **TOTAL** | **86** | **100** |

---

## Lo que hiciste bien

Lizbeth, se nota el trabajo: `DECISIONES.md` de 324 líneas, 10 JSON de ADF, 4 notebooks y 20 capturas con costo. La E2 está en código (`prueba_e2`) y el linked service del SQL sí está parametrizado.

- **Ingesta con ADF:** Lookup, ForEach{Copy+AppendVariable} y el linked service del SQL parametrizado.
- **Orquestación:** maestro correcto con el trigger 01:30 Lima en Stopped.
- **Bronce, plata, oro:** el job bronce→plata→oro terminó satisfactorio el 22-nov con los asserts de plata y oro, así que las 15 / 11 / 8 tablas quedan probadas.
- **Números de control:** los 7 cuadran y los verificaste con asserts en los notebooks.
- **Seguridad:** contraseña por Key Vault (`sqladmin-password`), Access Connector documentado y cero secretos en el repo.
- **Extensión propia:** E2 con `prueba_e2` en `e4_02_plata.py`: un registro sintético inválido dispara `AssertionError` y el job se detiene, con capturas de la prueba negativa y la positiva.
- **Decisiones:** un documento muy desarrollado, con 6 decisiones estructuradas (cargas diarias vs horarias, Managed Identities, serverless, medallón, E2 y E3).

---

## Para llevarlo al siguiente nivel

Lo que falta es concreto y rápido:

- **Repo y entrega:** el README no trae los nombres reales de tus recursos.
- **Ingesta con ADF:** el dataset del SQL solo tiene `tableName` (le faltan servidor y base), y escribes en `bronze/pedidos/fecha_carga=<fecha>` en vez de `bronze/pedidos_ingesta/<fecha>/<tabla>`. Funciona, pero se aparta de la convención del curso.
- **Orquestación:** el maestro no tiene user properties y tiene una sola annotation; y falta explicar por qué está pausado y cuál de los dos triggers manda.
- **Seguridad:** faltan el secret scope y el alcance de los roles.
  Para demostrarlo basta con esto y una captura de la salida (sin revelar ningún valor):

  ```python
  dbutils.secrets.listScopes()      # debe aparecer tu scope
  dbutils.secrets.list("tu-scope")  # aparecen los nombres, NUNCA los valores
  ```

---

## Mensaje final

Tu documentación es excelente, Lizbeth. Con los nombres reales en el README, el dataset parametrizado y las user properties en el maestro, el proyecto queda completo.

---

## Rúbrica aplicada

| # | Criterio | Pts | Qué se evalúa |
|---|---|---|---|
| 1 | Repo y entrega | 10 | Repo con docente invitado; README con nombres reales de recursos; `adf/`, `notebooks/`, `evidencia.md`, `DECISIONES.md`; captura de costo |
| 2 | Ingesta con ADF | 15 | `Lookup → ForEach → Copy + Append`; datasets SQL/CSV parametrizados; linked service del SQL parametrizado; `bronze/pedidos_ingesta/<fecha>/` con 5 tablas |
| 3 | Orquestación observable | 15 | Maestro `SetVar → Execute → Databricks Job → SetVar`; trigger 01:30 Lima con motivo; Monitor en Succeeded con user properties; annotations |
| 4 | Bronce, plata, oro (UC) | 15 | 15 bronce, 11 plata, 8 oro en `pede_dev`; notebooks en el repo; encadenados en el job diario con run Succeeded |
| 5 | Números de control | 15 | 1.425 pagados · 5.339.767,44 · 1.003.274,99 · 75 sin pago · 137/140 facturas · 333 checkout · 34.530 de 35.043 eventos |
| 6 | Seguridad y secretos | 10 | Contraseña solo en KV; LS con referencia; secret scope por KV; MSI con roles mínimos; ningún secreto en commits |
| 7 | Extensión propia | 15 | Funciona, código en el repo y prueba concreta en `evidencia.md` |
| 8 | Decisiones y defensa | 5 | Decisiones propias documentadas en `DECISIONES.md` |

---

*Revisión de proyecto final · P.E. Data Engineer · Azure (DSRP)*
