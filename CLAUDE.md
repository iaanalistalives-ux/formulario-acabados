# CLAUDE.md — Acabados (formulario-acabados)

Las reglas generales de LIVES están en el repo privado `lives-instrucciones` (CLAUDE.md). Aquí solo lo propio de esta herramienta.

## Qué hace
Eficiencia e incentivos del área de Acabados (Plancha, Vapor y Doblado), por módulo, persona, día y semana.
Encajado está excluido de todo el tablero: no se digita su producción.

## Pantallas
- `dashboard_acabados_v2.html` — Dashboard Acabados: KPI, gráficos, detalle por trabajador, semanas cerradas y vista Mensual/Anual. Solo lee.
  Botón "Exportar vista a Excel" con 4 hojas: RESUMEN, REGISTROS, MENSUAL_ANUAL y MIN_PROD_SEMANAL (minutos producidos y disponibles por semana y área, todas las semanas, con autofiltro; disponibles = Min Asist + Apoyo − Improductivos).
- `dashboard_acabados_v2.html` también lleva el botón **＋ Registrar** (desde el 05/11/2026): abre `formulario_acabados.html?embebido=1` en una ventana encima (modal, completa en celular). El botón consulta `acabados_puede_escribir_`; sin permiso sale gris "Solo para quien registra". Lo guardado queda en la semana abierta y se sigue al volver a entrar hasta Cerrar semana; el dashboard se actualiza solo 1,5 s después de cada guardado y al cerrar. Pide "¿Salir sin guardar?" si hay datos escritos. El formulario sigue teniendo su link directo.
- `formulario_acabados.html` — Digitación Acabados: producción por módulo, rechazo de auditoría, sistema DOB, asignar módulo, corregir/eliminar filas y Cerrar semana.
  Desde el 06/11/2026, debajo de "Integrantes de este módulo" sale el aviso "Con horas y sin módulo asignado" (acción `sinModulo` → `acabados_sin_modulo_()`, solo lee): personas de Plancha/Doblado con horas en una semana abierta un día sin tramo en `acabados_historial_modulos`. Ojo: `acabados_modulo_de` cuenta a esa persona en su módulo actual de la ficha, mientras "Integrantes" no la muestra; por eso un módulo podía calcular con una persona de más (caso DOB-M2 26/10/2026: 31% en vez de 48%).
  Desde el 10/11/2026 (rama `ausencias-descanso-medico`, aún sin publicar), sección "Descanso médico (persona que no vino)" debajo de los avisos: el digitador elige persona, desde, hasta y motivo y presiona Marcar. Ese día la persona queda con 0 minutos asistidos y 0 incentivo y no cuenta como integrante del módulo (no baja la eficiencia de sus compañeros); sus horas de HO no se borran. Lista de lo marcado con "Quitar". Aviso "Con horas, pero su módulo no tiene producción ese día" (acción `horasSinMarca`) para no olvidar marcar; no detecta a una persona ausente dentro de un módulo que sí produjo.
- `panel_acabados.html` — Panel de Acabados: carga de horas (HO), trabajadores, módulos, estándares por estilo, cliente y categoría.

## Base (Supabase lives-datos)
Una sola puerta: `acabados_api(p_accion, p)`, responde JSON. Es el contrato que replica Sistemas.
- Registros: `acabados_produccion`, `acabados_auditoria_dob`, `acabados_sistema_dob`, `acabados_horas`, `acabados_cargas_ho`.
- Catálogos: `acabados_trabajadores`, `acabados_modulos`, `acabados_historial_modulos`, `acabados_std_estilo`, `acabados_std_cliente`, `acabados_std_nivel`, `acabados_std_nivel_estilo`, `acabados_std_plancha_cat`, `acabados_curva` (eficiencia → % incentivo).
- Semanas: `acabados_semanas` (abierta/cerrada). Al cerrar se archiva en `acabados_hist_modulo_dia`, `acabados_hist_persona_dia` y `acabados_hist_persona_semana`.
- Ausencias (desde el 10/11/2026): `acabados_ausencias` (persona + fecha, una fila por día; `motivo_cod` → `acabados_motivos_ausencia`, hoy solo `DM` = Descanso médico). Va aparte de `acabados_horas`, así recargar HO no la borra. Solo se marca en semanas abiertas. `acabados_calc_modulos` y `acabados_calc_personas` ignoran esos días (todo lo demás —cierre de semana, dashboard, Excel— sale de ahí). Acciones de la puerta: `ausenciasListar`, `horasSinMarca` (leen), `ausenciaMarcar`, `ausenciaQuitar` (escriben, con bitácora). Registro: `A16 acabados ausencias descanso medico.sql`.
- Auditoría de cambios: `acabados_bitacora`, `acabados_papelera`.
- Cálculo: `acabados_calc_modulos` y `acabados_calc_personas` (semana abierta). Eficiencia = Min Prod ÷ (Min Asist + Apoyo − Improductivos), ponderada.
- Excel: `acabados_exportar_` (incluye `acabados_resumen_mensual_` y `acabados_min_prod_semanal_`).
- Semanas 29 a 37 vienen del Sheet antiguo: solo queda lo archivado. Las semanas 29 a 35 no guardaron el apoyo.
- Semana 36: se cerró dos veces en el Sheet y la carga tomó la copia en ceros; se recuperó del primer cierre el 21/10/2026.

## Quién la usa
- Ver: quien entra por el portal (lectura libre en Supabase).
- Registrar: `acabados_puede_escribir_()` — superusuario o quien tenga en la tabla `permisos` el formulario o el panel de Acabados. Es la pieza que Sistemas reemplaza con su seguridad.
- La responsable de Acabados digita, carga las horas y cierra la semana.

## Reglas propias
- Acabados no se toca en ningún cambio masivo (única excepción: modo oscuro).
- El Excel mantiene el formato de siempre en RESUMEN, REGISTROS y MENSUAL_ANUAL; no se pasa al bloque lives-excel-reporte. Solo la hoja MIN_PROD_SEMANAL usa el formato lives-excel.
