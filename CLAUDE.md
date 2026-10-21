# CLAUDE.md — Acabados (formulario-acabados)

Las reglas generales de LIVES están en el repo privado `lives-instrucciones` (CLAUDE.md). Aquí solo lo propio de esta herramienta.

## Qué hace
Eficiencia e incentivos del área de Acabados (Plancha, Vapor y Doblado), por módulo, persona, día y semana.
Encajado está excluido de todo el tablero: no se digita su producción.

## Pantallas
- `dashboard_acabados_v2.html` — Dashboard Acabados: KPI, gráficos, detalle por trabajador, semanas cerradas y vista Mensual/Anual. Solo lee.
  Botón "Exportar vista a Excel" con 4 hojas: RESUMEN, REGISTROS, MENSUAL_ANUAL y MIN_PROD_SEMANAL (minutos producidos por semana y área, todas las semanas, con autofiltro).
- `formulario_acabados.html` — Digitación Acabados: producción por módulo, rechazo de auditoría, sistema DOB, asignar módulo, corregir/eliminar filas y Cerrar semana.
- `panel_acabados.html` — Panel de Acabados: carga de horas (HO), trabajadores, módulos, estándares por estilo, cliente y categoría.

## Base (Supabase lives-datos)
Una sola puerta: `acabados_api(p_accion, p)`, responde JSON. Es el contrato que replica Sistemas.
- Registros: `acabados_produccion`, `acabados_auditoria_dob`, `acabados_sistema_dob`, `acabados_horas`, `acabados_cargas_ho`.
- Catálogos: `acabados_trabajadores`, `acabados_modulos`, `acabados_historial_modulos`, `acabados_std_estilo`, `acabados_std_cliente`, `acabados_std_nivel`, `acabados_std_nivel_estilo`, `acabados_std_plancha_cat`, `acabados_curva` (eficiencia → % incentivo).
- Semanas: `acabados_semanas` (abierta/cerrada). Al cerrar se archiva en `acabados_hist_modulo_dia`, `acabados_hist_persona_dia` y `acabados_hist_persona_semana`.
- Auditoría de cambios: `acabados_bitacora`, `acabados_papelera`.
- Cálculo: `acabados_calc_modulos` y `acabados_calc_personas` (semana abierta). Eficiencia = Min Prod ÷ (Min Asist + Apoyo − Improductivos), ponderada.
- Excel: `acabados_exportar_` (incluye `acabados_resumen_mensual_` y `acabados_min_prod_semanal_`).
- Semanas 29 a 37 vienen del Sheet antiguo: solo queda lo archivado. Las semanas 29 a 35 no guardaron el apoyo; la semana 36 no tiene producción archivada.

## Quién la usa
- Ver: quien entra por el portal (lectura libre en Supabase).
- Registrar: `acabados_puede_escribir_()` — superusuario, ctalepcio o quien tenga en la tabla `permisos` el formulario o el panel de Acabados. Es la pieza que Sistemas reemplaza con su seguridad.
- La responsable de Acabados digita, carga las horas y cierra la semana.

## Reglas propias
- Acabados no se toca en ningún cambio masivo (única excepción: modo oscuro).
- El Excel mantiene el formato de siempre en RESUMEN, REGISTROS y MENSUAL_ANUAL; no se pasa al bloque lives-excel-reporte. Solo la hoja MIN_PROD_SEMANAL usa el formato lives-excel.
