# PROMPT PARA GITHUB COPILOT AGENT

Quiero que analices completamente mi proyecto PyQt. El objetivo es
agregar la siguiente funcionalidad tipo Canva:

## 🎯 Objetivo

Implementar **Drag & Drop** desde el panel lateral de imágenes hacia el
canvas principal, de manera que un usuario pueda:

1.  Cargar imágenes desde el botón "Cargar Imágenes".
2.  Ver las miniaturas en el panel lateral (ya existente).
3.  Arrastrar cualquiera de esas miniaturas directamente hacia el
    canvas.
4.  Al soltarla, se debe crear un nuevo elemento/objeto imagen en el
    canvas, posicionado según el punto de drop.

------------------------------------------------------------------------

## 🔧 Requisitos técnicos específicos

### 1. Panel lateral

-   Convertir miniaturas en widgets que puedan emitir datos de drag.
-   MIME type: `application/x-image-path`
-   Payload: **ruta absoluta de la imagen**.

### 2. Canvas principal

-   Aceptar drops (`setAcceptDrops(True)`).
-   Implementar:
    -   `dragEnterEvent`
    -   `dragMoveEvent`
    -   `dropEvent`
-   Crear un objeto imagen en la posición donde el usuario suelta el
    mouse.

### 3. Objeto imagen en el canvas

-   Usar `QGraphicsScene` + `QGraphicsPixmapItem`, o clase custom
    dependiendo de la arquitectura actual.

------------------------------------------------------------------------

## 🧩 Comportamiento esperado

-   Feedback visual al arrastrar.
-   El canvas indica que acepta el drop.
-   La imagen aparece exactamente donde fue soltada.
-   Permitir múltiples imágenes.

------------------------------------------------------------------------

## 📚 Organización del código

-   Crear archivos nuevos si es necesario.
-   Mantener coherencia con la estructura actual del proyecto.
-   Comentar claramente las nuevas clases y métodos.

------------------------------------------------------------------------

## 🧪 Verificación

-   Mostrar diff completo de los cambios.
-   Explicar brevemente la arquitectura implementada.
-   Mantener compatibilidad con las funciones actuales.

------------------------------------------------------------------------

## 📝 Notas finales

-   Podés hacer refactor menor si es necesario.
-   Prioridad máxima: **funcionalidad estilo Canva 1:1**.
