# Panel-de-control-administrativo-de-aspirantes
Panel de control administrativo dentro de WordPress que permita al equipo de gestión del diplomado: visualizar y filtrar a los aspirantes en tiempo real, cambiar su estatus, subir y validar comprobantes de pago y documentos.

## Descripción del problema o necesidad
El diplomado necesitaba **una herramienta única** para:
- Ver todos los registros de Forminator en un solo lugar.
- Cambiar su estado a medida que avanzan en el proceso de admisión.
- Gestionar comprobantes de pago y enviarlos a los aspirantes.
- Evitar la carga manual de CSVs, correos separados y la pérdida de información.
Sin esa solución, el equipo gastaba horas cada semana en **exportar, limpiar y reenviar** datos.

## Propuesta de solución y alcance
| Aspecto | Solución propuesta |
|---------|-------------------|
| **Interfaz** | Página admin con pestañas (Aspirantes, Aceptados, Pago pendiente, Inscritos). |
| **Backend** | 7 endpoints AJAX seguros que operan directamente sobre la base de datos y la librería de medios. |
| **Flujo de trabajo** | 1) Cargar datos → 2) Filtrar/buscar → 3) Seleccionar → 4) Cambiar estado / subir comprobante / enviar email → 5) Exportar CSV. |
| **Alcance** | Cobertura completa del ciclo de vida de la inscripción *hasta* que el aspirante quede marcado como “inscrito”. No incluye **registro del formulario** (ya está en Forminator) ni **pagos reales** (solo gestión de comprobantes). |
| **Exclusiones** | No se implementa un gateway de pago ni integración con sistemas externos de facturación. |

## Cronograma y hitos principales
| Fase | Duración estimada | Hitos |
|------|-------------------|-------|
| **1. Requisitos y diseño** | 1 semana | Documento de flujo, mockups de UI (color palette premium, micro‑animaciones). |
| **2. Backend (PHP AJAX)** | 1 semana | Implementación de 7 handlers, pruebas unitarias. |
| **3. Frontend (JS + CSS)** | 1 semana | Tabla dinámica, modales, manejo de archivos, notificaciones. |
| **4. Integración y pruebas** | 1 semana | Pruebas de carga, validación de permisos, pruebas de envío de email. |
| **5. Deploy en staging** | 3 días | Instalación en entorno de pruebas, revisión de seguridad. |
| **6. Go‑Live** | 1 semana | Migración a producción, capacitación del equipo. |
| **7. Post‑lanzamiento** | 1 semana | Ajustes basados en feedback, documentación final. |

**Total estimado:** **≈ 5 semanas** de desarrollo completo.

---

## Objetivo de negocio
Crear **un panel de control administrativo** dentro de WordPress que permita al equipo de gestión del diplomado:

- Visualizar y filtrar a los aspirantes en tiempo real.
- Cambiar su **estatus** (aspirantes → aceptados → pago pendiente → inscritos).
- **Subir y validar comprobantes de pago**.
- Enviar **instrucciones bancarias** por correo a grupos seleccionados.
- Exportar datos a CSV y generar reportes rápidamente.

Todo ello reduce drásticamente el trabajo manual de extracción de datos, envío de correos y seguimiento de pagos, mejorando la **experiencia del equipo y la rapidez de admisión**.

## Fuentes y preparación de datos
| Fuente | Tipo | Proceso de extracción / limpieza |
|--------|------|---------------------------------|
| **Forminator** (`frmt_form_entry` y `frmt_form_entry_meta`) | Base de datos MySQL (WordPress) | Consulta mediante `$wpdb->get_results()`; se des‑serializan campos (`maybe_unserialize`) y se decodifican JSON cuando procede. |
| **Archivos subidos** (semblanzas, comprobantes) | Almacenamiento de WordPress (`wp_uploads`) | Se obtiene la URL y nombre mediante `wp_handle_upload`; se valida el MIME (`pdf, jpg, png`). |
| **Campos custom** (p. ej. `upload-1`, `select-1`, `comprobante_url`) | Metadatos del formulario | Se normalizan a texto plano (`sanitize_text_field`, `esc_url_raw`). |
| **Correo electrónico** (remitente, asunto, mensaje) | Input del admin | Se sanitizan (`sanitize_email`, `sanitize_text_field`). |
Los datos se convierten en un **array de objetos JavaScript** (`S.data`) con atributos como `id`, `nombre`, `apellido`, `email`, `status`, `sem_url`, `comp_url`, etc., listo para renderizar en la UI.

## Modelado y arquitectura
| Capa | Tecnologías / Herramientas | Comentario |
|------|---------------------------|------------|
| **Frontend** | HTML + **Vanilla JS** (ES6), CSS puro (estilos glass‑morphism, gradientes, micro‑animaciones) | UI reactiva, tabla filtrable, modales dinámicos, manejo de selección múltiple. |
| **Backend** | **PHP** (WordPress) → AJAX (`admin-ajax.php`) | 7 handlers: `get_aspirantes`, `update_aspirante_status`, `delete_aspirante`, `aspirantes_send_bank`, `aspirantes_upload_comprobante`, `aspirantes_debug_keys`, `aspirantes_debug_keys`. |
| **Base de datos** | MySQL (WordPress) | Acceso vía `$wpdb` con consultas preparadas para evitar inyección. |
| **Almacenamiento de ficheros** | WordPress Media Library (`wp_handle_upload`) | Validación MIME y tamaño ≤ 5 MB. |
| **Email** | `wp_mail()` | Envío de instrucciones bancarias con archivos adjuntos. |
| **Seguridad** | `check_ajax_referer`, `current_user_can('manage_options')` | Protección CSRF y de permisos. |

Arquitectura **cliente‑servidor sin frameworks**: el panel es una página admin de WordPress que carga JavaScript que llama a los endpoints PHP via `fetch` y actualiza la UI sin recargar.

<img width="1118" height="421" alt="image" src="https://github.com/user-attachments/assets/69ce6453-2924-48c9-a8bf-29d36715b641" />

## Despliegue e infraestructura
- **Entorno:** Servidor LAMP (Linux/Windows‑IIS + Apache/Nginx, PHP 7.4+, MySQL 5.7+).
- **Integración:** Código insertado como *snippet* o *mu‑plugin* en `wp-content/mu-plugins/` o mediante el editor de snippets.
- **Dependencias:** WordPress ≥ 5.8, Forminator plugin activo, permiso de administrador.
- **Instalación:** Copiar los archivos PHP al directorio indicado y activar la página admin mediante `add_menu_page`.
- **Escalabilidad:** Indexar `entry_id` y `meta_key` en la tabla de formularios; soporta miles de registros sin degradar.

---
## Resultados e impacto (valor tangible)
<img width="1173" height="608" alt="image" src="https://github.com/user-attachments/assets/58a487c8-2551-476e-b7c8-801f8be27d38" />


## Conclusión
El **Aspirantes Dashboard** entrega una solución integral, basada en WordPress, que automatiza la gestión de solicitudes, agiliza la recolección de comprobantes y permite una comunicación eficaz con los aspirantes. Con una arquitectura ligera (PHP + JS) y sin dependencias externas, el proyecto es **rápido de desplegar**, fácil de mantener y genera un impacto inmediato en la operatividad del equipo.




