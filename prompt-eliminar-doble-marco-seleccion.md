# Prompt: Eliminar Doble Marco de Selección en Items del Canvas

## 📋 CONTEXTO

**Usuario GitHub**: joseliviaa05-hub  
**Fecha**: 2025-11-15  
**Archivo**: `canvas_editor.py`  
**Problema**: Cuando selecciono una imagen en el canvas, aparecen DOS marcos:
1. Mi marco personalizado con handles (correcto)
2. Un marco exterior adicional de líneas punteadas (incorrecto)

## 🎯 OBJETIVO

Eliminar el marco de selección automático que PyQt6 dibuja por defecto, manteniendo únicamente los controles personalizados (handles y borde) que ya están implementados en el método `paint()`.

---

## 🔧 SOLUCIÓN REQUERIDA

### **PASO 1: Modificar `DraggableImageItem.paint()`**

Localiza la clase `DraggableImageItem` y su método `paint()`. Actualmente se ve así:

```python
def paint(self, painter, option, widget):
    # Dibujar la imagen
    super().paint(painter, option, widget)
    
    # Dibujar controles si está seleccionado
    if self.isSelected():
        rect = self.pixmap().rect()
        # ... código de handles ...
```

**CAMBIAR A**:

```python
def paint(self, painter, option, widget):
    """Dibujar imagen con controles personalizados (sin borde automático de Qt)"""
    
    # ===== SOLUCIÓN: Deshabilitar el borde de selección automático de Qt =====
    # Esta línea elimina el marco exterior que Qt dibuja automáticamente
    option.state &= ~QStyle.StateFlag.State_Selected
    
    # Dibujar la imagen
    super().paint(painter, option, widget)
    
    # Dibujar controles si está seleccionado
    if self.isSelected():
        rect = self.pixmap().rect()
        
        # Borde de selección animado
        pen = QPen(QColor(0, 120, 215), 2, Qt.PenStyle.DashLine)
        painter.setPen(pen)
        painter.setBrush(Qt.BrushStyle.NoBrush)
        painter.drawRect(rect)
        
        # Handles en las esquinas (para redimensionar proporcional)
        handle_size = self.resize_handle_size
        painter.setBrush(QBrush(QColor(0, 120, 215)))
        painter.setPen(QPen(QColor(255, 255, 255), 1))
        
        corners = [
            QRectF(rect.left() - handle_size/2, rect.top() - handle_size/2, handle_size, handle_size),  # TL
            QRectF(rect.right() - handle_size/2, rect.top() - handle_size/2, handle_size, handle_size),  # TR
            QRectF(rect.left() - handle_size/2, rect.bottom() - handle_size/2, handle_size, handle_size),  # BL
            QRectF(rect.right() - handle_size/2, rect.bottom() - handle_size/2, handle_size, handle_size),  # BR
        ]
        
        for corner in corners:
            painter.drawEllipse(corner)
        
        # Handles en los lados (para deformar)
        sides = [
            QRectF((rect.left() + rect.right())/2 - handle_size/2, rect.top() - handle_size/2, handle_size, handle_size),  # T
            QRectF((rect.left() + rect.right())/2 - handle_size/2, rect.bottom() - handle_size/2, handle_size, handle_size),  # B
            QRectF(rect.left() - handle_size/2, (rect.top() + rect.bottom())/2 - handle_size/2, handle_size, handle_size),  # L
            QRectF(rect.right() - handle_size/2, (rect.top() + rect.bottom())/2 - handle_size/2, handle_size, handle_size),  # R
        ]
        
        painter.setBrush(QBrush(QColor(255, 165, 0)))
        for side in sides:
            painter.drawRect(side)
```

**NOTA IMPORTANTE**: La línea clave es:
```python
option.state &= ~QStyle.StateFlag.State_Selected
```

Esta línea debe agregarse **ANTES** de `super().paint(painter, option, widget)`.

---

## 📝 EXPLICACIÓN TÉCNICA

### ¿Por qué aparecen dos marcos?

1. **Marco automático de Qt**: PyQt6 dibuja automáticamente un borde cuando un `QGraphicsItem` está seleccionado
2. **Marco personalizado**: Tu código dibuja otro borde en el método `paint()`

Resultado: **Doble marco superpuesto** que se ve poco profesional.

### ¿Qué hace `option.state &= ~QStyle.StateFlag.State_Selected`?

- `option.state` contiene el estado visual del item
- `QStyle.StateFlag.State_Selected` es el flag que indica "dibuja borde de selección"
- El operador `&= ~` **elimina** ese flag
- **Resultado**: Qt NO dibuja su marco automático, pero el item **sigue seleccionado funcionalmente**

---

## ✅ VERIFICACIÓN POST-IMPLEMENTACIÓN

Después de aplicar el cambio, verifica:

1. ✅ Al seleccionar una imagen, debe aparecer **UN SOLO** marco con handles
2. ✅ Los handles (círculos azules en esquinas, naranjas en lados) deben verse correctamente
3. ✅ La funcionalidad de redimensionar y rotar debe seguir funcionando
4. ✅ El marco debe aparecer SOLO cuando la imagen está seleccionada
5. ✅ Al deseleccionar, el marco debe desaparecer completamente

---

## 🎨 MEJORA OPCIONAL (BONUS)

Si deseas un look más moderno y profesional (estilo Figma/Canva), puedes cambiar los colores:

```python
def paint(self, painter, option, widget):
    option.state &= ~QStyle.StateFlag.State_Selected
    super().paint(painter, option, widget)
    
    if self.isSelected():
        rect = self.pixmap().rect()
        
        # Borde estilo Figma (azul brillante sólido)
        pen = QPen(QColor(24, 160, 251), 2, Qt.PenStyle.SolidLine)
        painter.setPen(pen)
        painter.setBrush(Qt.BrushStyle.NoBrush)
        painter.drawRect(rect)
        
        # Handles blancos con borde azul (más profesional)
        handle_size = self.resize_handle_size
        painter.setBrush(QBrush(QColor(255, 255, 255)))
        painter.setPen(QPen(QColor(24, 160, 251), 2))
        
        # Esquinas
        corners = [
            QRectF(rect.left() - handle_size/2, rect.top() - handle_size/2, handle_size, handle_size),
            QRectF(rect.right() - handle_size/2, rect.top() - handle_size/2, handle_size, handle_size),
            QRectF(rect.left() - handle_size/2, rect.bottom() - handle_size/2, handle_size, handle_size),
            QRectF(rect.right() - handle_size/2, rect.bottom() - handle_size/2, handle_size, handle_size),
        ]
        
        for corner in corners:
            painter.drawEllipse(corner)
        
        # Lados (mismo color para consistencia)
        sides = [
            QRectF((rect.left() + rect.right())/2 - handle_size/2, rect.top() - handle_size/2, handle_size, handle_size),
            QRectF((rect.left() + rect.right())/2 - handle_size/2, rect.bottom() - handle_size/2, handle_size, handle_size),
            QRectF(rect.left() - handle_size/2, (rect.top() + rect.bottom())/2 - handle_size/2, handle_size, handle_size),
            QRectF(rect.right() - handle_size/2, (rect.top() + rect.bottom())/2 - handle_size/2, handle_size, handle_size),
        ]
        
        for side in sides:
            painter.drawEllipse(side)  # Círculos en lugar de cuadrados
```

Esta mejora opcional hace que los handles se vean más modernos y consistentes.

---

## 🚨 IMPORTANTE

### NO modifiques:
- ❌ La funcionalidad de selección del item (`setFlag(ItemIsSelectable)`)
- ❌ Los event handlers (`mousePressEvent`, `mouseMoveEvent`, etc.)
- ❌ El método `isSelected()` - debe seguir funcionando igual
- ❌ La lógica de redimensionamiento y rotación

### SÍ modifica:
- ✅ **SOLO** el método `paint()` de `DraggableImageItem`
- ✅ Agrega la línea `option.state &= ~QStyle.StateFlag.State_Selected`
- ✅ Mantén todo el resto del código intacto

---

## 📊 RESULTADO ESPERADO

### ANTES (problema actual):
```
╔═══════════════════════════════╗  ← Marco automático de Qt (NO deseado)
║  ┌─────────────────────────┐  ║
║  │  [•]           [•]      │  ║  ← Tu marco personalizado (correcto)
║  │                         │  ║
║  │       IMAGEN            │  ║
║  │                         │  ║
║  │  [•]           [•]      │  ║
║  └─────────────────────────┘  ║
╚═══════════════════════════════╝
```

### DESPUÉS (resultado deseado):
```
┌─────────────────────────┐
│  [•]           [•]      │  ← Solo TU marco personalizado
│                         │
│       IMAGEN            │
│                         │
│  [•]           [•]      │
└─────────────────────────┘
```

---

## ✨ RESUMEN EJECUTIVO

**Acción requerida**: Agregar UNA SOLA LÍNEA de código en el método `paint()` de `DraggableImageItem`:

```python
option.state &= ~QStyle.StateFlag.State_Selected
```

Esta línea debe colocarse **INMEDIATAMENTE ANTES** de `super().paint(painter, option, widget)`.

**Efecto**: Elimina el marco de selección automático de Qt, dejando solo los controles personalizados.

**Impacto**: Mejora visual profesional, eliminando la duplicación de marcos.

**Riesgo**: Ninguno - no afecta la funcionalidad, solo el renderizado visual.

---

## 🎯 INSTRUCCIONES PARA EL AGENTE

1. ✅ Localiza el método `paint()` en la clase `DraggableImageItem` (aproximadamente línea 200-250)
2. ✅ Agrega la línea `option.state &= ~QStyle.StateFlag.State_Selected` ANTES de `super().paint()`
3. ✅ Verifica que el resto del método permanezca sin cambios
4. ✅ Guarda el archivo
5. ✅ NO modifiques ninguna otra parte del código

**Tiempo estimado**: 30 segundos  
**Dificultad**: Trivial  
**Prioridad**: ALTA (mejora visual inmediata)

---

**¡ADELANTE! Implementa este cambio simple que mejorará dramáticamente la apariencia del editor.**
