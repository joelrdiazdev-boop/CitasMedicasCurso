# Convenciones del proyecto — borrador para revisión

> Estado: propuesta. Este documento no describe todavía la estructura implementada; no autoriza por sí mismo una reorganización de carpetas.

## Propósito y alcance

Este repositorio contiene el MVP de citas médicas del curso. Usamos el curso como guía adaptable, no como una plantilla obligatoria. Organizamos el código de forma que cada historia se pueda implementar, comprender y verificar sin anticipar infraestructura ni funcionalidades futuras.

Estas convenciones orientan decisiones; si una historia justifica una excepción, la explicamos y actualizamos el documento antes de convertirla en regla general.

## Estado actual y organización prevista

La solución `CitasMedicas.slnx` contiene el proyecto `CitasMedicas.Web`. En este momento, el código publicado y la copia local tienen la misma estructura: `CitasMedicas.Web/Features/`, `CitasMedicas.Web/Pages/` y `CitasMedicas.Web/Shared/`, entre otras carpetas. Las subcarpetas `Shared/Data/`, `Shared/Models/` y `Shared/Services/` contienen solo archivos `.gitkeep`.

La organización prevista para el código de negocio, cuando las historias la requieran, es por límites de responsabilidad y funcionalidades dentro de cada límite. Un posible destino dentro de `CitasMedicas.Web/` es:

```text
Modules/
├── CatalogoMedico/
│   ├── Features/
│   │   └── ExplorarCatalogo/
│   └── Models/
├── Disponibilidad/
│   ├── Features/
│   │   └── ConsultarDisponibilidad/
│   └── Models/
└── Citas/
    ├── Features/
    │   ├── ReservarCita/
    │   ├── CancelarCita/
    │   └── ConsultarAgendaMedico/
    └── Models/

Data/
├── Configurations/
└── Seed/
```

El árbol es ilustrativo: no se crean carpetas vacías solo para completarlo ni se renombra código sin una tarea concreta. `Pages/` y los archivos de arranque existentes se mantienen donde correspondan hasta que un cambio justifique otra ubicación. Los nombres finales de carpetas y funcionalidades se decidirán al implementarlas, manteniendo consistencia con el código existente.

## Límites iniciales del negocio

- `CatalogoMedico` permite encontrar y seleccionar médicos, incluido el filtro por especialidad. No decide si un horario está libre ni si puede confirmarse una reserva.
- `Disponibilidad` presenta horarios consultables como libres u ocupados. Su calendario es una vista de la información disponible al consultarlo, no una garantía de reserva.
- `Citas` gestiona la reserva, los datos de contacto requeridos para ella, la cancelación y la agenda diaria del médico. Confirma la reserva y protege la regla de que un mismo horario no quede ocupado por dos citas vigentes del mismo médico.

Son límites iniciales, no tres módulos físicos que deban existir completos de inmediato. La captura de datos del paciente en la reserva no crea por sí sola un módulo `Pacientes`; la pantalla de agenda no crea un módulo `Dashboard`.

## Funcionalidades y modelos

Agrupamos el código específico de un caso de uso cerca de su funcionalidad. Un modelo que varias funcionalidades de un mismo límite necesitan puede vivir en `Models/` de ese límite; si solo sirve a una funcionalidad, puede permanecer dentro de ella.

No exigimos un controlador, servicio, interfaz o DTO por funcionalidad. Cada pieza debe tener una razón concreta, como hacer más claro el flujo o permitir una verificación útil. Evitamos duplicar reglas de negocio entre límites; cuando dos funcionalidades necesiten la misma información, identificamos primero quién es responsable de ella y acordamos cómo consultarla.

## Reserva, cancelación y concurrencia

Una cita `Reservada` ocupa un horario. Cancelarla cambia su estado a `Cancelada`, libera el horario y conserva el registro para que la agenda pueda mostrar la cancelación. Una nueva reserva del mismo médico y horario crea otra cita, no reutiliza ni borra la cancelada. El borrado lógico o físico de citas no forma parte del MVP.

`Disponibilidad` muestra el estado del calendario al consultarlo. `Citas` realiza la confirmación final con una garantía en persistencia que impide más de una cita vigente para un mismo médico y slot. Un `SELECT` previo puede orientar al usuario, pero no sustituye esa garantía ante solicitudes concurrentes. Si otro usuario obtuvo el horario antes, la segunda reserva no se confirma y se informa que debe elegir otro.

Para slots fijos, una posibilidad en PostgreSQL es un índice único parcial por médico e inicio de horario que incluya solo las citas que lo ocupan. Es una opción de implementación, no un esquema SQL aprobado. Al diseñar la persistencia definiremos la representación del slot, los estados que lo ocupan y cómo traducir el conflicto de unicidad a un resultado comprensible. Si se admiten duraciones variables o solapamientos, revisaremos la garantía antes de asumir que la igualdad de hora de inicio basta.

## Persistencia y código transversal

`Data/`, si se crea, aloja configuración técnica de persistencia y datos iniciales cuando sean necesarios; no es un límite de negocio ni un destino automático para todas las reglas y modelos. Las carpetas actuales `Shared/Data`, `Shared/Models` y `Shared/Services` no son la estructura objetivo solo por existir.

`Shared/` se reserva para una responsabilidad genuinamente transversal y justificada. Que dos funcionalidades consulten información sobre citas no convierte a `Cita` en un modelo compartido: su propiedad de negocio sigue en `Citas`. No se crean abstracciones o servicios compartidos de manera preventiva.

## Alcance del MVP y verificación

Las historias del repositorio orientan el trabajo, pero sus criterios deben mantenerse coherentes con el alcance acordado. En esta iteración posponemos la confirmación por correo de la reserva y de la cancelación. En particular, US-05 y el criterio de email de US-06 siguen pendientes hasta que actualicemos explícitamente su planificación; no se consideran implementados ni se marcan como completados.

Implementamos lo necesario para la historia en curso, comprobamos los criterios que sí están dentro de su alcance y declaramos lo que no se haya podido probar. No cambiamos límites, estructura o alcance silenciosamente: explicamos el motivo y actualizamos estas convenciones si la decisión se mantiene.
