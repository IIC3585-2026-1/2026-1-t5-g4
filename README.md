# Trabajo 5: Frameworks Vue y Svelte - Grupo 4

Este repositorio contiene dos versiones de la aplicación **MyUnits**:
1. **MyUnitsVue:** Un conversor exhaustivo y detallado, manejando múltiples selectores, prefijos métricos y estado complejo.
2. **MyUnitsSvelte (Express):** Una versión ágil enfocada en tarjetas de conversión rápida (presets), aprovechando la reactividad nativa de Svelte.

## Bono de Originalidad Implementado

Para la evaluación del bono de originalidad, incluimos funcionalidades extra en **ambas** aplicaciones:

* **Historial Reactivo:** Las últimas 5 conversiones se guardan y muestran automáticamente en pantalla, actualizándose en tiempo real sin necesidad de recargar ni presionar botones de cálculo.
* **Copia Rápida al Portapapeles:** Implementación del API `navigator.clipboard` para copiar los resultados con un solo clic, incluyendo manejo de temporizadores para el feedback visual (cambio de estado a "Copiado").
* **Unidades Completas:** Se incluyeron todas las unidades requeridas por la rúbrica (Longitud, Área, Volumen, Masa y Temperatura) adaptadas a la filosofía de cada framework.
* **Modo Oscuro Nativo:** Sistema de temas claro/oscuro utilizando variables CSS dinámicas en ambas arquitecturas.

## Instrucciones de ejecución
Para cada proyecto, ingresar a la carpeta respectiva y ejecutar:
1. `npm install`
2. `npm run dev`