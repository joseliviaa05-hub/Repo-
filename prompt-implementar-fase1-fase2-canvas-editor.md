# Prompt: Implementación Completa FASE 1 y FASE 2 - Canvas Editor Profesional

## 📋 CONTEXTO DEL PROYECTO

**Usuario GitHub**: joseliviaa05-hub  
**Fecha**: 2025-11-15  
**Proyecto**: Editor de Canvas Profesional en Python (PyQt6)  
**Archivo principal**: `canvas_editor.py`

### Estado Actual del Código
Tengo un editor de canvas funcional con:
- ✅ Sistema de capas y z-index
- ✅ Drag & drop de imágenes
- ✅ Redimensionamiento con handles
- ✅ Rotación y transformaciones básicas
- ✅ Undo/Redo con historial JSON
- ✅ Exportación a PDF/PNG/JPG/TIFF
- ✅ Plantillas personalizables
- ✅ Sistema de zoom y navegación

### Problemas Identificados que DEBES Corregir
1. ❌ Gestión de memoria con archivos temporales (no se limpian)
2. ❌ Error en `toggle_grid`: `Qt.CheckState.Checked.value` → debe ser `Qt.CheckState.Checked`
3. ❌ Falta try-catch robusto en `restore_state` para imágenes eliminadas
4. ❌ `wheelEvent` no funciona correctamente (debe estar en el QGraphicsView)
5. ❌ `snap_to_grid` se setea pero nunca se implementa
6. ❌ Multi-selección no actualiza panel de propiedades correctamente
7. ❌ Copiar/pegar no preserva posiciones relativas entre objetos

---

## 🎯 OBJETIVO PRINCIPAL

Implementar **COMPLETA Y FUNCIONALMENTE** las siguientes fases de desarrollo, corrigiendo TODOS los bugs identificados y agregando TODAS las funcionalidades especificadas.

---

## 🚀 FASE 1: FUNCIONALIDAD CORE (2-3 semanas)

### **1.1 CORRECCIÓN DE BUGS CRÍTICOS**

#### Bug #1: Gestión de Memoria - Archivos Temporales
```python
"""
PROBLEMA ACTUAL:
En flip_selected_horizontal() y flip_selected_vertical() se crean archivos
temporales en /temp/ pero nunca se eliminan.

SOLUCIÓN REQUERIDA:
1. Crear clase TempFileManager para gestionar archivos temporales
2. Registrar todos los archivos temporales creados
3. Eliminarlos al cerrar la aplicación
4. Implementar cleanup automático cada X minutos
5. Usar context managers (with statement) para garantizar limpieza
"""

class TempFileManager:
    """
    Gestor centralizado de archivos temporales
    
    Características requeridas:
    - Singleton pattern
    - Lista de archivos temporales activos
    - Método cleanup() para eliminar todos
    - Auto-cleanup en __del__
    - Método register_temp_file(path)
    - Método cleanup_old_files(max_age_hours=24)
    """
    pass

# USAR en flip_selected_horizontal(), flip_selected_vertical() y cualquier
# operación que cree archivos temporales
```

#### Bug #2: Error en toggle_grid
```python
"""
LÍNEA ACTUAL (INCORRECTA):
self.show_grid = state == Qt.CheckState.Checked.value

CAMBIAR A:
self.show_grid = state == Qt.CheckState.Checked

O MEJOR AÚN:
self.show_grid = bool(state)
"""
```

#### Bug #3: Manejo Robusto en restore_state
```python
"""
PROBLEMA:
Si una imagen del historial fue eliminada o movida, restore_state() falla
completamente y el usuario pierde todo el trabajo.

SOLUCIÓN:
1. Envolver Image.open() en try-except
2. Si falla, mostrar placeholder o skip esa imagen
3. Registrar error en log pero continuar restaurando otras imágenes
4. Notificar al usuario qué imágenes no se pudieron cargar
5. Ofrecer opción de "buscar imagen manualmente"
"""

def restore_state(self, state_json: str):
    try:
        state = json.loads(state_json)
        failed_images = []
        
        for img_data in state['images']:
            try:
                # Intentar cargar imagen
                if not os.path.exists(img_data['path']):
                    failed_images.append(img_data['path'])
                    continue
                
                # ... resto del código
                
            except Exception as e:
                logging.error(f"Error restaurando imagen {img_data.get('path')}: {e}")
                failed_images.append(img_data.get('path', 'desconocida'))
                continue
        
        if failed_images:
            self.show_missing_images_dialog(failed_images)
            
    except Exception as e:
        QMessageBox.critical(self, "Error Crítico", 
                           f"No se pudo restaurar el estado:\n{e}")
```

#### Bug #4: Zoom con Ctrl+Scroll
```python
"""
PROBLEMA:
wheelEvent está en QMainWindow, no captura eventos del QGraphicsView

SOLUCIÓN:
1. Subclasificar QGraphicsView → CustomGraphicsView
2. Implementar wheelEvent ahí
3. O instalar eventFilter específico en el view.viewport()
"""

class CustomGraphicsView(QGraphicsView):
    def __init__(self, scene, parent=None):
        super().__init__(scene, parent)
        self.canvas_editor = parent
        
    def wheelEvent(self, event):
        if event.modifiers() & Qt.KeyboardModifier.ControlModifier:
            # Zoom hacia la posición del cursor
            delta = event.angleDelta().y()
            zoom_factor = 1.15 if delta > 0 else 1/1.15
            
            # Guardar posición del mouse en la escena
            old_pos = self.mapToScene(event.position().toPoint())
            
            # Aplicar zoom
            self.scale(zoom_factor, zoom_factor)
            
            # Ajustar para mantener punto bajo cursor
            new_pos = self.mapToScene(event.position().toPoint())
            delta = new_pos - old_pos
            self.translate(delta.x(), delta.y())
            
            # Actualizar label de zoom en editor
            if self.canvas_editor:
                self.canvas_editor.update_zoom_label()
            
            event.accept()
        else:
            super().wheelEvent(event)
```

#### Bug #5: Snap to Grid - Implementación Real
```python
"""
ACTUALMENTE: self.snap_to_grid se setea pero no hace nada

IMPLEMENTAR:
En itemChange() de DraggableImageItem, snap a la grid cuando esté habilitado
"""

def itemChange(self, change, value):
    if change == QGraphicsItem.GraphicsItemChange.ItemPositionChange:
        if self.canvas_editor.snap_to_grid and not self.isSelected():
            # Snap a grid
            grid_size_cm = 1.0  # 1 cm
            grid_size_px = cm_to_pixels(grid_size_cm, self.canvas_editor.canvas_dpi)
            
            new_pos = value  # QPointF
            snapped_x = round(new_pos.x() / grid_size_px) * grid_size_px
            snapped_y = round(new_pos.y() / grid_size_px) * grid_size_px
            
            return QPointF(snapped_x, snapped_y)
    
    # ... resto del código original
    return super().itemChange(change, value)
```

#### Bug #6: Multi-selección en Panel de Propiedades
```python
"""
PROBLEMA:
update_properties_from_selection() solo funciona con 1 objeto

SOLUCIÓN:
Mostrar propiedades comunes cuando hay múltiples seleccionados
"""

def update_properties_from_selection(self):
    selected = [item for item in self.scene.selectedItems() 
                if isinstance(item, DraggableImageItem)]
    
    if len(selected) == 0:
        # Deshabilitar panel
        self.disable_properties_panel()
        
    elif len(selected) == 1:
        # Código actual (funciona)
        canvas_item = selected[0].canvas_item
        # ... código existente
        
    else:  # Múltiple selección
        # Mostrar solo propiedades comunes
        self.enable_properties_panel()
        
        # Calcular promedio o mostrar "Múltiple" si difieren
        avg_x = sum(item.canvas_item.x for item in selected) / len(selected)
        avg_y = sum(item.canvas_item.y for item in selected) / len(selected)
        avg_opacity = sum(item.canvas_item.opacity for item in selected) / len(selected)
        
        # Verificar si todas tienen la misma rotación
        rotations = [item.canvas_item.rotation for item in selected]
        same_rotation = all(r == rotations[0] for r in rotations)
        
        self.prop_x.setValue(avg_x)
        self.prop_y.setValue(avg_y)
        self.prop_opacity.setValue(int(avg_opacity * 100))
        
        if same_rotation:
            self.prop_rotation.setValue(int(rotations[0]))
        else:
            self.prop_rotation.setSpecialValueText("Múltiple")
        
        # Deshabilitar width/height para múltiple selección
        self.prop_width.setEnabled(False)
        self.prop_height.setEnabled(False)
```

#### Bug #7: Copiar/Pegar Preservando Posiciones Relativas
```python
"""
PROBLEMA:
Al copiar 3 imágenes alineadas y pegar, se desalinean

SOLUCIÓN:
Calcular offset relativo al centro de la selección
"""

def copy_selected(self):
    selected = [item for item in self.scene.selectedItems() 
                if isinstance(item, DraggableImageItem)]
    
    if not selected:
        return
    
    # Calcular centroide de la selección
    center_x = sum(item.canvas_item.x for item in selected) / len(selected)
    center_y = sum(item.canvas_item.y for item in selected) / len(selected)
    
    self.clipboard_items = []
    for item in selected:
        # Guardar posición RELATIVA al centro
        offset_x = item.canvas_item.x - center_x
        offset_y = item.canvas_item.y - center_y
        
        copied = CanvasImageItem(
            image_path=item.canvas_item.image_path,
            x=offset_x,  # RELATIVO
            y=offset_y,  # RELATIVO
            width=item.canvas_item.width,
            height=item.canvas_item.height,
            rotation=item.canvas_item.rotation,
            z_index=item.canvas_item.z_index,
            opacity=item.canvas_item.opacity,
            original_aspect_ratio=item.canvas_item.original_aspect_ratio
        )
        self.clipboard_items.append(copied)
    
    self.clipboard_center = (center_x, center_y)  # Guardar para paste
    self.statusBar().showMessage(f"{len(selected)} imagen(es) copiada(s)", 2000)

def paste_from_clipboard(self):
    if not self.clipboard_items:
        return
    
    # Pegar en nueva posición (offset +2cm del centro original)
    paste_center_x = self.clipboard_center[0] + 2.0
    paste_center_y = self.clipboard_center[1] + 2.0
    
    for canvas_item in self.clipboard_items:
        # Calcular posición absoluta
        new_x = paste_center_x + canvas_item.x  # x es relativo
        new_y = paste_center_y + canvas_item.y  # y es relativo
        
        new_item = CanvasImageItem(
            image_path=canvas_item.image_path,
            x=new_x,
            y=new_y,
            width=canvas_item.width,
            height=canvas_item.height,
            rotation=canvas_item.rotation,
            z_index=len(self.canvas_images),
            opacity=canvas_item.opacity,
            original_aspect_ratio=canvas_item.original_aspect_ratio
        )
        
        # ... resto del código para agregar al canvas
```

---

### **1.2 HERRAMIENTA DE TEXTO COMPLETA**

Esta es la característica MÁS IMPORTANTE que falta. Debe ser PROFESIONAL.

#### Clase Base: TextCanvasItem
```python
@dataclass
class TextCanvasItem:
    """
    Objeto de texto en el canvas con todas sus propiedades
    
    REQUERIMIENTOS MÍNIMOS:
    """
    text: str
    x: float
    y: float
    width: float  # Ancho de la caja de texto
    height: float  # Alto (auto-ajustable)
    
    # Tipografía
    font_family: str = "Arial"
    font_size: float = 16.0  # En puntos
    font_weight: str = "normal"  # normal, bold, light, black
    font_style: str = "normal"  # normal, italic, oblique
    
    # Color y apariencia
    color: str = "#000000"
    background_color: str = "transparent"
    background_opacity: float = 0.0
    
    # Alineación y espaciado
    alignment: str = "left"  # left, center, right, justify
    line_height: float = 1.2  # Múltiplo del tamaño de fuente
    letter_spacing: float = 0.0  # En puntos
    
    # Decoración
    underline: bool = False
    strikethrough: bool = False
    text_transform: str = "none"  # none, uppercase, lowercase, capitalize
    
    # Efectos
    shadow_enabled: bool = False
    shadow_offset_x: float = 2.0
    shadow_offset_y: float = 2.0
    shadow_blur: float = 4.0
    shadow_color: str = "#00000080"
    
    outline_enabled: bool = False
    outline_width: float = 1.0
    outline_color: str = "#000000"
    
    # Transformación
    rotation: float = 0
    opacity: float = 1.0
    z_index: int = 0
    
    # Estado
    locked: bool = False
    visible: bool = True
    editable: bool = True  # Si está en modo edición
    
    uuid: str = field(default_factory=lambda: str(uuid.uuid4()))
```

#### Clase Gráfica: DraggableTextItem
```python
class DraggableTextItem(QGraphicsTextItem):
    """
    Item de texto editable, arrastrable y estilizable
    
    CARACTERÍSTICAS REQUERIDAS:
    - Doble click para entrar en modo edición
    - Edición in-place con cursor parpadeante
    - Selección de texto con mouse
    - Rich text (negrita, cursiva en partes del texto)
    - Auto-resize vertical cuando el texto crece
    - Handles para redimensionar ancho de caja
    - Mismo sistema de transformación que imágenes
    - Copy/paste de texto formateado
    """
    
    def __init__(self, text_item: TextCanvasItem, canvas_editor, parent=None):
        super().__init__(parent)
        self.text_item = text_item
        self.canvas_editor = canvas_editor
        
        # Configuración inicial
        self.setPlainText(text_item.text)
        self.setDefaultTextColor(QColor(text_item.color))
        self.setTextInteractionFlags(Qt.TextInteractionFlag.NoTextInteraction)
        
        # Aplicar fuente
        font = QFont(text_item.font_family, int(text_item.font_size))
        font.setBold(text_item.font_weight == "bold")
        font.setItalic(text_item.font_style == "italic")
        font.setUnderline(text_item.underline)
        font.setStrikeOut(text_item.strikethrough)
        self.setFont(font)
        
        # Alineación
        cursor = self.textCursor()
        text_format = QTextBlockFormat()
        if text_item.alignment == "center":
            text_format.setAlignment(Qt.AlignmentFlag.AlignCenter)
        elif text_item.alignment == "right":
            text_format.setAlignment(Qt.AlignmentFlag.AlignRight)
        elif text_item.alignment == "justify":
            text_format.setAlignment(Qt.AlignmentFlag.AlignJustify)
        else:
            text_format.setAlignment(Qt.AlignmentFlag.AlignLeft)
        cursor.mergeBlockFormat(text_format)
        
        # Transformaciones
        self.setFlag(QGraphicsItem.GraphicsItemFlag.ItemIsMovable, not text_item.locked)
        self.setFlag(QGraphicsItem.GraphicsItemFlag.ItemIsSelectable)
        self.setFlag(QGraphicsItem.GraphicsItemFlag.ItemSendsGeometryChanges)
        self.setOpacity(text_item.opacity)
        self.setZValue(text_item.z_index)
        self.setRotation(text_item.rotation)
        
        # Tamaño de caja
        self.setTextWidth(cm_to_pixels(text_item.width, canvas_editor.canvas_dpi))
        
        # Estado
        self.is_editing = False
        self.resize_handle_size = 10
    
    def mouseDoubleClickEvent(self, event):
        """Doble click → Modo edición"""
        if not self.text_item.locked and event.button() == Qt.MouseButton.LeftButton:
            self.enter_edit_mode()
            event.accept()
        else:
            super().mouseDoubleClickEvent(event)
    
    def enter_edit_mode(self):
        """Activar edición de texto"""
        self.is_editing = True
        self.setTextInteractionFlags(
            Qt.TextInteractionFlag.TextEditorInteraction
        )
        self.setFocus()
        
        # Seleccionar todo el texto
        cursor = self.textCursor()
        cursor.select(QTextCursor.SelectionType.Document)
        self.setTextCursor(cursor)
        
        # Cambiar cursor del item
        self.setCursor(Qt.CursorShape.IBeamCursor)
        
        # Notificar al editor
        self.canvas_editor.statusBar().showMessage("Modo edición - Presiona Esc para salir", 0)
    
    def exit_edit_mode(self):
        """Salir de modo edición"""
        self.is_editing = False
        self.setTextInteractionFlags(Qt.TextInteractionFlag.NoTextInteraction)
        self.clearFocus()
        self.setCursor(Qt.CursorShape.ArrowCursor)
        
        # Actualizar texto en text_item
        self.text_item.text = self.toPlainText()
        
        # Actualizar altura si cambió
        doc_height_px = self.document().size().height()
        self.text_item.height = pixels_to_cm(doc_height_px, self.canvas_editor.canvas_dpi)
        
        self.canvas_editor.save_history_state()
        self.canvas_editor.statusBar().showMessage("Texto actualizado", 2000)
    
    def keyPressEvent(self, event):
        """Manejar teclas en modo edición"""
        if self.is_editing:
            if event.key() == Qt.Key.Key_Escape:
                self.exit_edit_mode()
                event.accept()
                return
            # Permitir edición normal
            super().keyPressEvent(event)
        else:
            # Pasar evento al canvas
            event.ignore()
    
    def focusOutEvent(self, event):
        """Salir de edición al perder foco"""
        if self.is_editing:
            self.exit_edit_mode()
        super().focusOutEvent(event)
    
    def paint(self, painter, option, widget):
        # Dibujar efectos si están habilitados
        if self.text_item.shadow_enabled:
            self.draw_text_shadow(painter)
        
        if self.text_item.outline_enabled:
            self.draw_text_outline(painter)
        
        # Dibujar fondo si está habilitado
        if self.text_item.background_color != "transparent":
            painter.save()
            painter.setBrush(QBrush(QColor(self.text_item.background_color)))
            painter.setOpacity(self.text_item.background_opacity)
            painter.drawRect(self.boundingRect())
            painter.restore()
        
        # Dibujar texto
        super().paint(painter, option, widget)
        
        # Dibujar borde de selección y handles
        if self.isSelected():
            self.draw_selection_border(painter)
    
    def draw_text_shadow(self, painter):
        """Dibujar sombra de texto"""
        painter.save()
        painter.translate(
            self.text_item.shadow_offset_x,
            self.text_item.shadow_offset_y
        )
        painter.setPen(QPen(QColor(self.text_item.shadow_color)))
        # ... implementar blur si es necesario
        painter.restore()
    
    def draw_text_outline(self, painter):
        """Dibujar contorno de texto"""
        # Implementar usando QPainterPath
        pass
    
    def draw_selection_border(self, painter):
        """Dibujar borde y handles cuando está seleccionado"""
        rect = self.boundingRect()
        
        # Borde
        pen = QPen(QColor(0, 120, 215), 2, Qt.PenStyle.DashLine)
        painter.setPen(pen)
        painter.drawRect(rect)
        
        # Handles para redimensionar ancho
        handle_size = self.resize_handle_size
        painter.setBrush(QBrush(QColor(0, 120, 215)))
        painter.setPen(QPen(QColor(255, 255, 255), 1))
        
        # Handle derecho (para cambiar ancho)
        right_handle = QRectF(
            rect.right() - handle_size/2,
            rect.center().y() - handle_size/2,
            handle_size, handle_size
        )
        painter.drawEllipse(right_handle)
    
    def contextMenuEvent(self, event):
        """Menú contextual para texto"""
        menu = QMenu()
        
        edit_action = menu.addAction("✏️ Editar Texto")
        menu.addSeparator()
        
        # Estilo
        style_menu = menu.addMenu("🎨 Estilo")
        bold_action = style_menu.addAction("Bold")
        italic_action = style_menu.addAction("Italic")
        underline_action = style_menu.addAction("Subrayado")
        
        menu.addSeparator()
        
        # Alineación
        align_menu = menu.addMenu("⬌ Alineación")
        align_left = align_menu.addAction("⬅️ Izquierda")
        align_center = align_menu.addAction("↔️ Centro")
        align_right = align_menu.addAction("➡️ Derecha")
        align_justify = align_menu.addAction("⬌ Justificar")
        
        menu.addSeparator()
        
        # Efectos
        effects_menu = menu.addMenu("✨ Efectos")
        shadow_action = effects_menu.addAction("🌑 Sombra...")
        outline_action = effects_menu.addAction("⭕ Contorno...")
        
        menu.addSeparator()
        
        duplicate_action = menu.addAction("📋 Duplicar")
        delete_action = menu.addAction("🗑️ Eliminar")
        
        action = menu.exec(event.screenPos())
        
        if action == edit_action:
            self.enter_edit_mode()
        elif action == bold_action:
            self.toggle_bold()
        elif action == italic_action:
            self.toggle_italic()
        # ... implementar resto de acciones
```

#### Herramienta de Texto en la UI
```python
"""
AGREGAR EN EL PANEL IZQUIERDO (después de cargar imágenes):
"""

def setup_text_tool_ui(self):
    """Panel de herramientas de texto"""
    text_group = QGroupBox("✏️ Herramientas de Texto")
    text_layout = QVBoxLayout()
    
    # Botón para agregar texto
    add_text_btn = QPushButton("➕ Agregar Texto")
    add_text_btn.clicked.connect(self.add_text_to_canvas)
    add_text_btn.setStyleSheet("""
        QPushButton {
            background: #4CAF50;
            color: white;
            font-weight: bold;
            padding: 10px;
            border-radius: 5px;
        }
        QPushButton:hover {
            background: #45a049;
        }
    """)
    
    # Propiedades de texto (cuando hay texto seleccionado)
    text_props_widget = QWidget()
    text_props_layout = QFormLayout()
    
    self.text_font_combo = QFontComboBox()
    self.text_font_combo.currentFontChanged.connect(self.update_text_font)
    
    self.text_size_spin = QSpinBox()
    self.text_size_spin.setRange(6, 200)
    self.text_size_spin.setValue(16)
    self.text_size_spin.setSuffix(" pt")
    self.text_size_spin.valueChanged.connect(self.update_text_size)
    
    self.text_color_btn = QPushButton("Color")
    self.text_color_btn.clicked.connect(self.choose_text_color)
    
    # Botones de estilo
    style_layout = QHBoxLayout()
    
    self.text_bold_btn = QPushButton("B")
    self.text_bold_btn.setCheckable(True)
    self.text_bold_btn.setFont(QFont("Arial", 10, QFont.Weight.Bold))
    self.text_bold_btn.clicked.connect(self.toggle_text_bold)
    
    self.text_italic_btn = QPushButton("I")
    self.text_italic_btn.setCheckable(True)
    font = QFont("Arial", 10)
    font.setItalic(True)
    self.text_italic_btn.setFont(font)
    self.text_italic_btn.clicked.connect(self.toggle_text_italic)
    
    self.text_underline_btn = QPushButton("U")
    self.text_underline_btn.setCheckable(True)
    font = QFont("Arial", 10)
    font.setUnderline(True)
    self.text_underline_btn.setFont(font)
    self.text_underline_btn.clicked.connect(self.toggle_text_underline)
    
    style_layout.addWidget(self.text_bold_btn)
    style_layout.addWidget(self.text_italic_btn)
    style_layout.addWidget(self.text_underline_btn)
    
    # Alineación
    align_layout = QHBoxLayout()
    
    self.text_align_left_btn = QPushButton("⬅️")
    self.text_align_left_btn.clicked.connect(lambda: self.set_text_alignment("left"))
    
    self.text_align_center_btn = QPushButton("↔️")
    self.text_align_center_btn.clicked.connect(lambda: self.set_text_alignment("center"))
    
    self.text_align_right_btn = QPushButton("➡️")
    self.text_align_right_btn.clicked.connect(lambda: self.set_text_alignment("right"))
    
    align_layout.addWidget(self.text_align_left_btn)
    align_layout.addWidget(self.text_align_center_btn)
    align_layout.addWidget(self.text_align_right_btn)
    
    text_props_layout.addRow("Fuente:", self.text_font_combo)
    text_props_layout.addRow("Tamaño:", self.text_size_spin)
    text_props_layout.addRow("Color:", self.text_color_btn)
    text_props_layout.addRow("Estilo:", style_layout)
    text_props_layout.addRow("Alineación:", align_layout)
    
    text_props_widget.setLayout(text_props_layout)
    text_props_widget.setEnabled(False)  # Habilitar cuando haya texto seleccionado
    self.text_props_widget = text_props_widget
    
    text_layout.addWidget(add_text_btn)
    text_layout.addWidget(text_props_widget)
    text_group.setLayout(text_layout)
    
    return text_group

def add_text_to_canvas(self):
    """Agregar nuevo texto al canvas"""
    # Posición central del canvas
    x_cm = self.canvas_width_cm / 2 - 5
    y_cm = self.canvas_height_cm / 2 - 1
    
    text_item = TextCanvasItem(
        text="Doble click para editar",
        x=x_cm,
        y=y_cm,
        width=10.0,  # 10 cm de ancho
        height=2.0,
        font_size=24.0,
        z_index=len(self.canvas_images) + len(self.text_items)
    )
    
    graphic_text = DraggableTextItem(text_item, self)
    x_px = cm_to_pixels(x_cm, self.canvas_dpi)
    y_px = cm_to_pixels(y_cm, self.canvas_dpi)
    graphic_text.setPos(x_px, y_px)
    
    self.scene.addItem(graphic_text)
    self.text_items.append(text_item)
    
    # Auto-entrar en modo edición
    graphic_text.enter_edit_mode()
    
    self.save_history_state()
    self.statusBar().showMessage("Texto agregado - Doble click para editar", 3000)
```

---

### **1.3 FORMAS GEOMÉTRICAS BÁSICAS**

Implementar rectángulos, círculos, líneas con propiedades editables.

#### Clase Base: ShapeCanvasItem
```python
@dataclass
class ShapeCanvasItem:
    """
    Forma geométrica en el canvas
    
    TIPOS SOPORTADOS:
    - rectangle
    - ellipse (círculo/elipse)
    - line
    - arrow
    - triangle
    - polygon (n lados)
    - star (n puntas)
    """
    shape_type: str  # "rectangle", "ellipse", "line", etc.
    x: float
    y: float
    width: float
    height: float
    
    # Estilo
    fill_color: str = "#3498db"
    fill_opacity: float = 1.0
    stroke_color: str = "#2c3e50"
    stroke_width: float = 2.0
    stroke_style: str = "solid"  # solid, dashed, dotted
    
    # Para rectángulos
    corner_radius: float = 0.0  # Radio de esquinas redondeadas
    
    # Para polígonos y estrellas
    sides: int = 5
    star_inner_radius: float = 0.5  # Ratio para estrellas
    
    # Para líneas y flechas
    line_start_x: float = 0.0
    line_start_y: float = 0.0
    line_end_x: float = 0.0
    line_end_y: float = 0.0
    arrow_head_size: float = 10.0
    
    # Transformación
    rotation: float = 0
    opacity: float = 1.0
    z_index: int = 0
    
    # Estado
    locked: bool = False
    visible: bool = True
    
    uuid: str = field(default_factory=lambda: str(uuid.uuid4()))
```

#### Clase Gráfica: DraggableShapeItem
```python
class DraggableShapeItem(QGraphicsPathItem):
    """
    Item de forma geométrica editable
    
    CARACTERÍSTICAS:
    - Redimensionable con handles
    - Cambiar color de relleno y borde
    - Radio de esquinas ajustable (rectángulos)
    - Convertir entre tipos de forma
    """
    
    def __init__(self, shape_item: ShapeCanvasItem, canvas_editor, parent=None):
        super().__init__(parent)
        self.shape_item = shape_item
        self.canvas_editor = canvas_editor
        
        # Crear path según el tipo
        self.update_path()
        
        # Estilo
        self.update_style()
        
        # Transformaciones
        self.setFlag(QGraphicsItem.GraphicsItemFlag.ItemIsMovable, not shape_item.locked)
        self.setFlag(QGraphicsItem.GraphicsItemFlag.ItemIsSelectable)
        self.setFlag(QGraphicsItem.GraphicsItemFlag.ItemSendsGeometryChanges)
        self.setOpacity(shape_item.opacity)
        self.setZValue(shape_item.z_index)
        self.setRotation(shape_item.rotation)
    
    def update_path(self):
        """Actualizar path según tipo de forma"""
        path = QPainterPath()
        
        if self.shape_item.shape_type == "rectangle":
            rect = QRectF(0, 0, 
                         cm_to_pixels(self.shape_item.width, self.canvas_editor.canvas_dpi),
                         cm_to_pixels(self.shape_item.height, self.canvas_editor.canvas_dpi))
            
            if self.shape_item.corner_radius > 0:
                radius = cm_to_pixels(self.shape_item.corner_radius, self.canvas_editor.canvas_dpi)
                path.addRoundedRect(rect, radius, radius)
            else:
                path.addRect(rect)
        
        elif self.shape_item.shape_type == "ellipse":
            rect = QRectF(0, 0,
                         cm_to_pixels(self.shape_item.width, self.canvas_editor.canvas_dpi),
                         cm_to_pixels(self.shape_item.height, self.canvas_editor.canvas_dpi))
            path.addEllipse(rect)
        
        elif self.shape_item.shape_type == "triangle":
            w = cm_to_pixels(self.shape_item.width, self.canvas_editor.canvas_dpi)
            h = cm_to_pixels(self.shape_item.height, self.canvas_editor.canvas_dpi)
            path.moveTo(w/2, 0)
            path.lineTo(w, h)
            path.lineTo(0, h)
            path.closeSubpath()
        
        elif self.shape_item.shape_type == "star":
            self.create_star_path(path)
        
        elif self.shape_item.shape_type == "polygon":
            self.create_polygon_path(path)
        
        self.setPath(path)
    
    def create_star_path(self, path):
        """Crear path de estrella"""
        import math
        
        w = cm_to_pixels(self.shape_item.width, self.canvas_editor.canvas_dpi)
        h = cm_to_pixels(self.shape_item.height, self.canvas_editor.canvas_dpi)
        cx, cy = w/2, h/2
        outer_r = min(w, h) / 2
        inner_r = outer_r * self.shape_item.star_inner_radius
        points = self.shape_item.sides
        
        angle = math.pi / points
        
        for i in range(points * 2):
            r = outer_r if i % 2 == 0 else inner_r
            theta = i * angle - math.pi / 2
            x = cx + r * math.cos(theta)
            y = cy + r * math.sin(theta)
            
            if i == 0:
                path.moveTo(x, y)
            else:
                path.lineTo(x, y)
        
        path.closeSubpath()
    
    def create_polygon_path(self, path):
        """Crear path de polígono regular"""
        import math
        
        w = cm_to_pixels(self.shape_item.width, self.canvas_editor.canvas_dpi)
        h = cm_to_pixels(self.shape_item.height, self.canvas_editor.canvas_dpi)
        cx, cy = w/2, h/2
        radius = min(w, h) / 2
        sides = self.shape_item.sides
        
        angle = 2 * math.pi / sides
        
        for i in range(sides):
            theta = i * angle - math.pi / 2
            x = cx + radius * math.cos(theta)
            y = cy + radius * math.sin(theta)
            
            if i == 0:
                path.moveTo(x, y)
            else:
                path.lineTo(x, y)
        
        path.closeSubpath()
    
    def update_style(self):
        """Actualizar estilo de la forma"""
        # Relleno
        fill = QBrush(QColor(self.shape_item.fill_color))
        self.setBrush(fill)
        
        # Borde
        pen = QPen(QColor(self.shape_item.stroke_color), self.shape_item.stroke_width)
        
        if self.shape_item.stroke_style == "dashed":
            pen.setStyle(Qt.PenStyle.DashLine)
        elif self.shape_item.stroke_style == "dotted":
            pen.setStyle(Qt.PenStyle.DotLine)
        else:
            pen.setStyle(Qt.PenStyle.SolidLine)
        
        self.setPen(pen)
    
    def paint(self, painter, option, widget):
        super().paint(painter, option, widget)
        
        # Dibujar handles si está seleccionado
        if self.isSelected():
            self.draw_resize_handles(painter)
    
    def draw_resize_handles(self, painter):
        """Dibujar handles de redimensionamiento"""
        rect = self.boundingRect()
        handle_size = 10
        
        painter.setBrush(QBrush(QColor(0, 120, 215)))
        painter.setPen(QPen(QColor(255, 255, 255), 1))
        
        # Esquinas
        corners = [
            QPointF(rect.left(), rect.top()),
            QPointF(rect.right(), rect.top()),
            QPointF(rect.left(), rect.bottom()),
            QPointF(rect.right(), rect.bottom()),
        ]
        
        for corner in corners:
            handle = QRectF(
                corner.x() - handle_size/2,
                corner.y() - handle_size/2,
                handle_size, handle_size
            )
            painter.drawEllipse(handle)
```

#### UI para Crear Formas
```python
def setup_shapes_tool_ui(self):
    """Panel de herramientas de formas"""
    shapes_group = QGroupBox("🔷 Formas Geométricas")
    shapes_layout = QVBoxLayout()
    
    # Grid de botones de formas
    shapes_grid = QGridLayout()
    
    rect_btn = QPushButton("⬜ Rectángulo")
    rect_btn.clicked.connect(lambda: self.add_shape_to_canvas("rectangle"))
    
    circle_btn = QPushButton("⭕ Círculo")
    circle_btn.clicked.connect(lambda: self.add_shape_to_canvas("ellipse"))
    
    triangle_btn = QPushButton("🔺 Triángulo")
    triangle_btn.clicked.connect(lambda: self.add_shape_to_canvas("triangle"))
    
    star_btn = QPushButton("⭐ Estrella")
    star_btn.clicked.connect(lambda: self.add_shape_to_canvas("star"))
    
    polygon_btn = QPushButton("⬡ Polígono")
    polygon_btn.clicked.connect(lambda: self.add_shape_to_canvas("polygon"))
    
    line_btn = QPushButton("➖ Línea")
    line_btn.clicked.connect(lambda: self.add_shape_to_canvas("line"))
    
    shapes_grid.addWidget(rect_btn, 0, 0)
    shapes_grid.addWidget(circle_btn, 0, 1)
    shapes_grid.addWidget(triangle_btn, 1, 0)
    shapes_grid.addWidget(star_btn, 1, 1)
    shapes_grid.addWidget(polygon_btn, 2, 0)
    shapes_grid.addWidget(line_btn, 2, 1)
    
    # Propiedades de forma (cuando hay forma seleccionada)
    shape_props_widget = QWidget()
    shape_props_layout = QFormLayout()
    
    self.shape_fill_color_btn = QPushButton("Color Relleno")
    self.shape_fill_color_btn.clicked.connect(self.choose_shape_fill_color)
    
    self.shape_stroke_color_btn = QPushButton("Color Borde")
    self.shape_stroke_color_btn.clicked.connect(self.choose_shape_stroke_color)
    
    self.shape_stroke_width_spin = QDoubleSpinBox()
    self.shape_stroke_width_spin.setRange(0, 20)
    self.shape_stroke_width_spin.setValue(2)
    self.shape_stroke_width_spin.setSuffix(" px")
    self.shape_stroke_width_spin.valueChanged.connect(self.update_shape_stroke_width)
    
    self.shape_corner_radius_spin = QDoubleSpinBox()
    self.shape_corner_radius_spin.setRange(0, 10)
    self.shape_corner_radius_spin.setValue(0)
    self.shape_corner_radius_spin.setSuffix(" cm")
    self.shape_corner_radius_spin.valueChanged.connect(self.update_shape_corner_radius)
    
    shape_props_layout.addRow("Relleno:", self.shape_fill_color_btn)
    shape_props_layout.addRow("Borde:", self.shape_stroke_color_btn)
    shape_props_layout.addRow("Grosor:", self.shape_stroke_width_spin)
    shape_props_layout.addRow("Redondez:", self.shape_corner_radius_spin)
    
    shape_props_widget.setLayout(shape_props_layout)
    shape_props_widget.setEnabled(False)
    self.shape_props_widget = shape_props_widget
    
    shapes_layout.addLayout(shapes_grid)
    shapes_layout.addWidget(shape_props_widget)
    shapes_group.setLayout(shapes_layout)
    
    return shapes_group

def add_shape_to_canvas(self, shape_type):
    """Agregar nueva forma al canvas"""
    x_cm = self.canvas_width_cm / 2 - 2.5
    y_cm = self.canvas_height_cm / 2 - 2.5
    
    shape_item = ShapeCanvasItem(
        shape_type=shape_type,
        x=x_cm,
        y=y_cm,
        width=5.0,
        height=5.0,
        z_index=len(self.canvas_images) + len(self.text_items) + len(self.shape_items)
    )
    
    graphic_shape = DraggableShapeItem(shape_item, self)
    x_px = cm_to_pixels(x_cm, self.canvas_dpi)
    y_px = cm_to_pixels(y_cm, self.canvas_dpi)
    graphic_shape.setPos(x_px, y_px)
    
    self.scene.addItem(graphic_shape)
    self.shape_items.append(shape_item)
    
    self.save_history_state()
    self.statusBar().showMessage(f"Forma {shape_type} agregada", 2000)
```

---

### **1.4 SISTEMA DE ALINEACIÓN BÁSICO**

Herramientas para alinear y distribuir objetos.

```python
class AlignmentTools:
    """
    Herramientas de alineación de objetos
    """
    
    def __init__(self, canvas_editor):
        self.editor = canvas_editor
    
    def align_left(self):
        """Alinear objetos al borde izquierdo"""
        selected = self.editor.get_selected_items()
        if len(selected) < 2:
            QMessageBox.information(self.editor, "Info", 
                                  "Selecciona al menos 2 objetos para alinear")
            return
        
        # Encontrar el left mínimo
        min_left = min(item.pos().x() for item in selected)
        
        for item in selected:
            item.setPos(min_left, item.pos().y())
            self.update_canvas_item_position(item)
        
        self.editor.save_history_state()
        self.editor.statusBar().showMessage("Objetos alineados a la izquierda", 2000)
    
    def align_center_horizontal(self):
        """Alinear objetos al centro horizontal"""
        selected = self.editor.get_selected_items()
        if len(selected) < 2:
            return
        
        # Calcular centro promedio
        avg_center_x = sum(item.pos().x() + item.boundingRect().width()/2 
                          for item in selected) / len(selected)
        
        for item in selected:
            new_x = avg_center_x - item.boundingRect().width()/2
            item.setPos(new_x, item.pos().y())
            self.update_canvas_item_position(item)
        
        self.editor.save_history_state()
        self.editor.statusBar().showMessage("Objetos centrados horizontalmente", 2000)
    
    def align_right(self):
        """Alinear objetos al borde derecho"""
        selected = self.editor.get_selected_items()
        if len(selected) < 2:
            return
        
        # Encontrar el right máximo
        max_right = max(item.pos().x() + item.boundingRect().width() 
                       for item in selected)
        
        for item in selected:
            new_x = max_right - item.boundingRect().width()
            item.setPos(new_x, item.pos().y())
            self.update_canvas_item_position(item)
        
        self.editor.save_history_state()
    
    def align_top(self):
        """Alinear objetos al borde superior"""
        selected = self.editor.get_selected_items()
        if len(selected) < 2:
            return
        
        min_top = min(item.pos().y() for item in selected)
        
        for item in selected:
            item.setPos(item.pos().x(), min_top)
            self.update_canvas_item_position(item)
        
        self.editor.save_history_state()
    
    def align_center_vertical(self):
        """Alinear objetos al centro vertical"""
        selected = self.editor.get_selected_items()
        if len(selected) < 2:
            return
        
        avg_center_y = sum(item.pos().y() + item.boundingRect().height()/2 
                          for item in selected) / len(selected)
        
        for item in selected:
            new_y = avg_center_y - item.boundingRect().height()/2
            item.setPos(item.pos().x(), new_y)
            self.update_canvas_item_position(item)
        
        self.editor.save_history_state()
    
    def align_bottom(self):
        """Alinear objetos al borde inferior"""
        selected = self.editor.get_selected_items()
        if len(selected) < 2:
            return
        
        max_bottom = max(item.pos().y() + item.boundingRect().height() 
                        for item in selected)
        
        for item in selected:
            new_y = max_bottom - item.boundingRect().height()
            item.setPos(item.pos().x(), new_y)
            self.update_canvas_item_position(item)
        
        self.editor.save_history_state()
    
    def distribute_horizontal(self):
        """Distribuir objetos horizontalmente con espaciado uniforme"""
        selected = self.editor.get_selected_items()
        if len(selected) < 3:
            QMessageBox.information(self.editor, "Info", 
                                  "Selecciona al menos 3 objetos para distribuir")
            return
        
        # Ordenar por posición X
        sorted_items = sorted(selected, key=lambda item: item.pos().x())
        
        # Calcular espacio total
        left_most = sorted_items[0].pos().x()
        right_most = sorted_items[-1].pos().x() + sorted_items[-1].boundingRect().width()
        total_width = sum(item.boundingRect().width() for item in sorted_items)
        available_space = right_most - left_most - total_width
        spacing = available_space / (len(sorted_items) - 1)
        
        # Distribuir
        current_x = left_most
        for item in sorted_items:
            item.setPos(current_x, item.pos().y())
            self.update_canvas_item_position(item)
            current_x += item.boundingRect().width() + spacing
        
        self.editor.save_history_state()
        self.editor.statusBar().showMessage("Objetos distribuidos horizontalmente", 2000)
    
    def distribute_vertical(self):
        """Distribuir objetos verticalmente con espaciado uniforme"""
        selected = self.editor.get_selected_items()
        if len(selected) < 3:
            return
        
        sorted_items = sorted(selected, key=lambda item: item.pos().y())
        
        top_most = sorted_items[0].pos().y()
        bottom_most = sorted_items[-1].pos().y() + sorted_items[-1].boundingRect().height()
        total_height = sum(item.boundingRect().height() for item in sorted_items)
        available_space = bottom_most - top_most - total_height
        spacing = available_space / (len(sorted_items) - 1)
        
        current_y = top_most
        for item in sorted_items:
            item.setPos(item.pos().x(), current_y)
            self.update_canvas_item_position(item)
            current_y += item.boundingRect().height() + spacing
        
        self.editor.save_history_state()
    
    def update_canvas_item_position(self, graphic_item):
        """Actualizar posición en el canvas_item correspondiente"""
        if isinstance(graphic_item, DraggableImageItem):
            graphic_item.canvas_item.x = pixels_to_cm(graphic_item.pos().x(), 
                                                      self.editor.canvas_dpi)
            graphic_item.canvas_item.y = pixels_to_cm(graphic_item.pos().y(), 
                                                      self.editor.canvas_dpi)
        elif isinstance(graphic_item, DraggableTextItem):
            graphic_item.text_item.x = pixels_to_cm(graphic_item.pos().x(), 
                                                    self.editor.canvas_dpi)
            graphic_item.text_item.y = pixels_to_cm(graphic_item.pos().y(), 
                                                    self.editor.canvas_dpi)
        elif isinstance(graphic_item, DraggableShapeItem):
            graphic_item.shape_item.x = pixels_to_cm(graphic_item.pos().x(), 
                                                     self.editor.canvas_dpi)
            graphic_item.shape_item.y = pixels_to_cm(graphic_item.pos().y(), 
                                                     self.editor.canvas_dpi)

# Agregar al CanvasEditor:
def setup_alignment_ui(self):
    """Panel de alineación"""
    align_group = QGroupBox("⬌ Alineación")
    align_layout = QVBoxLayout()
    
    # Alineación horizontal
    h_align_layout = QHBoxLayout()
    h_align_layout.addWidget(QLabel("Horizontal:"))
    
    align_left_btn = QPushButton("⬅️")
    align_left_btn.setToolTip("Alinear a la izquierda")
    align_left_btn.clicked.connect(self.alignment_tools.align_left)
    
    align_center_h_btn = QPushButton("↔️")
    align_center_h_btn.setToolTip("Centrar horizontalmente")
    align_center_h_btn.clicked.connect(self.alignment_tools.align_center_horizontal)
    
    align_right_btn = QPushButton("➡️")
    align_right_btn.setToolTip("Alinear a la derecha")
    align_right_btn.clicked.connect(self.alignment_tools.align_right)
    
    h_align_layout.addWidget(align_left_btn)
    h_align_layout.addWidget(align_center_h_btn)
    h_align_layout.addWidget(align_right_btn)
    
    # Alineación vertical
    v_align_layout = QHBoxLayout()
    v_align_layout.addWidget(QLabel("Vertical:"))
    
    align_top_btn = QPushButton("⬆️")
    align_top_btn.setToolTip("Alinear arriba")
    align_top_btn.clicked.connect(self.alignment_tools.align_top)
    
    align_center_v_btn = QPushButton("↕️")
    align_center_v_btn.setToolTip("Centrar verticalmente")
    align_center_v_btn.clicked.connect(self.alignment_tools.align_center_vertical)
    
    align_bottom_btn = QPushButton("⬇️")
    align_bottom_btn.setToolTip("Alinear abajo")
    align_bottom_btn.clicked.connect(self.alignment_tools.align_bottom)
    
    v_align_layout.addWidget(align_top_btn)
    v_align_layout.addWidget(align_center_v_btn)
    v_align_layout.addWidget(align_bottom_btn)
    
    # Distribución
    dist_layout = QHBoxLayout()
    dist_layout.addWidget(QLabel("Distribuir:"))
    
    dist_h_btn = QPushButton("⬌ H")
    dist_h_btn.setToolTip("Distribuir horizontalmente")
    dist_h_btn.clicked.connect(self.alignment_tools.distribute_horizontal)
    
    dist_v_btn = QPushButton("⬍ V")
    dist_v_btn.setToolTip("Distribuir verticalmente")
    dist_v_btn.clicked.connect(self.alignment_tools.distribute_vertical)
    
    dist_layout.addWidget(dist_h_btn)
    dist_layout.addWidget(dist_v_btn)
    
    align_layout.addLayout(h_align_layout)
    align_layout.addLayout(v_align_layout)
    align_layout.addLayout(dist_layout)
    align_group.setLayout(align_layout)
    
    return align_group

# En __init__ del CanvasEditor:
self.alignment_tools = AlignmentTools(self)
```

---

## 🎨 FASE 2: MEJORAS VISUALES (2-3 semanas)

### **2.1 FILTROS DE IMAGEN BÁSICOS**

Sistema completo de ajustes de imagen usando PIL.

```python
class ImageFilters:
    """
    Sistema de filtros y ajustes de imagen
    
    Usar PIL/Pillow para aplicar filtros
    """
    
    @staticmethod
    def adjust_brightness(image: Image.Image, value: float) -> Image.Image:
        """
        Ajustar brillo
        value: -1.0 a 1.0 (0 = sin cambio)
        """
        enhancer = ImageEnhance.Brightness(image)
        # Convertir -1..1 a 0..2
        factor = 1.0 + value
        return enhancer.enhance(factor)
    
    @staticmethod
    def adjust_contrast(image: Image.Image, value: float) -> Image.Image:
        """
        Ajustar contraste
        value: -1.0 a 1.0
        """
        enhancer = ImageEnhance.Contrast(image)
        factor = 1.0 + value
        return enhancer.enhance(factor)
    
    @staticmethod
    def adjust_saturation(image: Image.Image, value: float) -> Image.Image:
        """
        Ajustar saturación
        value: -1.0 a 1.0
        """
        enhancer = ImageEnhance.Color(image)
        factor = 1.0 + value
        return enhancer.enhance(factor)
    
    @staticmethod
    def adjust_sharpness(image: Image.Image, value: float) -> Image.Image:
        """
        Ajustar nitidez
        value: -1.0 a 1.0
        """
        enhancer = ImageEnhance.Sharpness(image)
        factor = 1.0 + value
        return enhancer.enhance(factor)
    
    @staticmethod
    def apply_blur(image: Image.Image, radius: float) -> Image.Image:
        """
        Aplicar desenfoque gaussiano
        radius: 0 a 50
        """
        return image.filter(ImageFilter.GaussianBlur(radius))
    
    @staticmethod
    def apply_sharpen(image: Image.Image) -> Image.Image:
        """Aplicar filtro de enfoque"""
        return image.filter(ImageFilter.SHARPEN)
    
    @staticmethod
    def convert_grayscale(image: Image.Image) -> Image.Image:
        """Convertir a escala de grises"""
        return image.convert('L').convert('RGB')
    
    @staticmethod
    def apply_sepia(image: Image.Image) -> Image.Image:
        """Aplicar efecto sepia"""
        grayscale = image.convert('L')
        sepia = Image.new('RGB', image.size)
        
        for y in range(image.height):
            for x in range(image.width):
                gray = grayscale.getpixel((x, y))
                r = int(gray * 0.9)
                g = int(gray * 0.7)
                b = int(gray * 0.4)
                sepia.putpixel((x, y), (r, g, b))
        
        return sepia
    
    @staticmethod
    def invert_colors(image: Image.Image) -> Image.Image:
        """Invertir colores"""
        from PIL import ImageOps
        return ImageOps.invert(image.convert('RGB'))
    
    @staticmethod
    def apply_vignette(image: Image.Image, strength: float = 0.5) -> Image.Image:
        """
        Aplicar viñeta (oscurecer bordes)
        strength: 0.0 a 1.0
        """
        import numpy as np
        from PIL import Image
        
        # Convertir a numpy array
        img_array = np.array(image)
        
        # Crear máscara de viñeta
        rows, cols = img_array.shape[:2]
        X_resultant_kernel = cv2.getGaussianKernel(cols, cols * strength)
        Y_resultant_kernel = cv2.getGaussianKernel(rows, rows * strength)
        
        kernel = Y_resultant_kernel * X_resultant_kernel.T
        mask = kernel / kernel.max()
        
        # Aplicar máscara
        for i in range(3):  # RGB
            img_array[:,:,i] = img_array[:,:,i] * mask
        
        return Image.fromarray(img_array.astype('uint8'))

# Diálogo para ajustar filtros:
class ImageFiltersDialog(QDialog):
    """
    Diálogo modal para ajustar filtros de imagen en tiempo real
    """
    
    def __init__(self, image_item: DraggableImageItem, parent=None):
        super().__init__(parent)
        self.image_item = image_item
        self.canvas_editor = parent
        self.original_image = Image.open(image_item.canvas_item.image_path)
        self.preview_image = self.original_image.copy()
        
        # Estado de filtros
        self.filters_state = {
            'brightness': 0.0,
            'contrast': 0.0,
            'saturation': 0.0,
            'sharpness': 0.0,
            'blur': 0.0,
        }
        
        self.setWindowTitle("🎨 Ajustar Imagen")
        self.resize(800, 600)
        self.setup_ui()
    
    def setup_ui(self):
        layout = QVBoxLayout()
        
        # Layout horizontal: preview + controles
        content_layout = QHBoxLayout()
        
        # === PREVIEW (Izquierda) ===
        preview_widget = QWidget()
        preview_layout = QVBoxLayout()
        
        self.preview_label = QLabel()
        self.preview_label.setAlignment(Qt.AlignmentFlag.AlignCenter)
        self.preview_label.setStyleSheet("""
            QLabel {
                border: 2px solid #ccc;
                background: #f0f0f0;
                min-width: 400px;
                min-height: 400px;
            }
        """)
        
        # Botones de antes/después
        compare_layout = QHBoxLayout()
        self.show_original_btn = QPushButton("👁️ Ver Original")
        self.show_original_btn.pressed.connect(self.show_original_preview)
        self.show_original_btn.released.connect(self.show_filtered_preview)
        compare_layout.addWidget(self.show_original_btn)
        
        preview_layout.addWidget(QLabel("<b>Vista Previa</b>"))
        preview_layout.addWidget(self.preview_label)
        preview_layout.addLayout(compare_layout)
        preview_widget.setLayout(preview_layout)
        
        # === CONTROLES (Derecha) ===
        controls_widget = QWidget()
        controls_layout = QVBoxLayout()
        
        # Scroll area para controles
        scroll = QScrollArea()
        scroll.setWidgetResizable(True)
        scroll.setMinimumWidth(350)
        
        scroll_content = QWidget()
        scroll_layout = QVBoxLayout()
        
        # BRILLO
        brightness_group = QGroupBox("☀️ Brillo")
        brightness_layout = QVBoxLayout()
        self.brightness_slider = QSlider(Qt.Orientation.Horizontal)
        self.brightness_slider.setRange(-100, 100)
        self.brightness_slider.setValue(0)
        self.brightness_slider.valueChanged.connect(self.on_brightness_changed)
        self.brightness_value_label = QLabel("0")
        brightness_layout.addWidget(self.brightness_slider)
        brightness_layout.addWidget(self.brightness_value_label)
        brightness_group.setLayout(brightness_layout)
        
        # CONTRASTE
        contrast_group = QGroupBox("◐ Contraste")
        contrast_layout = QVBoxLayout()
        self.contrast_slider = QSlider(Qt.Orientation.Horizontal)
        self.contrast_slider.setRange(-100, 100)
        self.contrast_slider.setValue(0)
        self.contrast_slider.valueChanged.connect(self.on_contrast_changed)
        self.contrast_value_label = QLabel("0")
        contrast_layout.addWidget(self.contrast_slider)
        contrast_layout.addWidget(self.contrast_value_label)
        contrast_group.setLayout(contrast_layout)
        
        # SATURACIÓN
        saturation_group = QGroupBox("🎨 Saturación")
        saturation_layout = QVBoxLayout()
        self.saturation_slider = QSlider(Qt.Orientation.Horizontal)
        self.saturation_slider.setRange(-100, 100)
        self.saturation_slider.setValue(0)
        self.saturation_slider.valueChanged.connect(self.on_saturation_changed)
        self.saturation_value_label = QLabel("0")
        saturation_layout.addWidget(self.saturation_slider)
        saturation_layout.addWidget(self.saturation_value_label)
        saturation_group.setLayout(saturation_layout)
        
        # NITIDEZ
        sharpness_group = QGroupBox("🔍 Nitidez")
        sharpness_layout = QVBoxLayout()
        self.sharpness_slider = QSlider(Qt.Orientation.Horizontal)
        self.sharpness_slider.setRange(-100, 100)
        self.sharpness_slider.setValue(0)
        self.sharpness_slider.valueChanged.connect(self.on_sharpness_changed)
        self.sharpness_value_label = QLabel("0")
        sharpness_layout.addWidget(self.sharpness_slider)
        sharpness_layout.addWidget(self.sharpness_value_label)
        sharpness_group.setLayout(sharpness_layout)
        
        # DESENFOQUE
        blur_group = QGroupBox("🌫️ Desenfoque")
        blur_layout = QVBoxLayout()
        self.blur_slider = QSlider(Qt.Orientation.Horizontal)
        self.blur_slider.setRange(0, 50)
        self.blur_slider.setValue(0)
        self.blur_slider.valueChanged.connect(self.on_blur_changed)
        self.blur_value_label = QLabel("0")
        blur_layout.addWidget(self.blur_slider)
        blur_layout.addWidget(self.blur_value_label)
        blur_group.setLayout(blur_layout)
        
        # FILTROS RÁPIDOS
        quick_filters_group = QGroupBox("⚡ Filtros Rápidos")
        quick_filters_layout = QVBoxLayout()
        
        grayscale_btn = QPushButton("⚫ Blanco y Negro")
        grayscale_btn.clicked.connect(self.apply_grayscale)
        
        sepia_btn = QPushButton("🟤 Sepia")
        sepia_btn.clicked.connect(self.apply_sepia)
        
        invert_btn = QPushButton("🔄 Invertir Colores")
        invert_btn.clicked.connect(self.apply_invert)
        
        vignette_btn = QPushButton("🌑 Viñeta")
        vignette_btn.clicked.connect(self.apply_vignette)
        
        quick_filters_layout.addWidget(grayscale_btn)
        quick_filters_layout.addWidget(sepia_btn)
        quick_filters_layout.addWidget(invert_btn)
        quick_filters_layout.addWidget(vignette_btn)
        quick_filters_group.setLayout(quick_filters_layout)
        
        # Botón de resetear
        reset_btn = QPushButton("🔄 Restablecer Todo")
        reset_btn.clicked.connect(self.reset_all_filters)
        reset_btn.setStyleSheet("background: #ff5722; color: white; font-weight: bold; padding: 8px;")
        
        # Agregar todos los grupos
        scroll_layout.addWidget(brightness_group)
        scroll_layout.addWidget(contrast_group)
        scroll_layout.addWidget(saturation_group)
        scroll_layout.addWidget(sharpness_group)
        scroll_layout.addWidget(blur_group)
        scroll_layout.addWidget(quick_filters_group)
        scroll_layout.addWidget(reset_btn)
        scroll_layout.addStretch()
        
        scroll_content.setLayout(scroll_layout)
        scroll.setWidget(scroll_content)
        
        controls_layout.addWidget(QLabel("<b>Ajustes</b>"))
        controls_layout.addWidget(scroll)
        controls_widget.setLayout(controls_layout)
        
        # Agregar al layout principal
        content_layout.addWidget(preview_widget, 1)
        content_layout.addWidget(controls_widget)
        
        # Botones de acción
        buttons = QDialogButtonBox(
            QDialogButtonBox.StandardButton.Save | 
            QDialogButtonBox.StandardButton.Cancel
        )
        buttons.accepted.connect(self.accept)
        buttons.rejected.connect(self.reject)
        
        layout.addLayout(content_layout)
        layout.addWidget(buttons)
        
        self.setLayout(layout)
        
        # Mostrar preview inicial
        self.update_preview()
    
    def on_brightness_changed(self, value):
        self.filters_state['brightness'] = value / 100.0
        self.brightness_value_label.setText(str(value))
        self.apply_filters()
    
    def on_contrast_changed(self, value):
        self.filters_state['contrast'] = value / 100.0
        self.contrast_value_label.setText(str(value))
        self.apply_filters()
    
    def on_saturation_changed(self, value):
        self.filters_state['saturation'] = value / 100.0
        self.saturation_value_label.setText(str(value))
        self.apply_filters()
    
    def on_sharpness_changed(self, value):
        self.filters_state['sharpness'] = value / 100.0
        self.sharpness_value_label.setText(str(value))
        self.apply_filters()
    
    def on_blur_changed(self, value):
        self.filters_state['blur'] = value
        self.blur_value_label.setText(str(value))
        self.apply_filters()
    
    def apply_filters(self):
        """Aplicar todos los filtros activos"""
        img = self.original_image.copy()
        
        # Aplicar ajustes básicos
        if self.filters_state['brightness'] != 0:
            img = ImageFilters.adjust_brightness(img, self.filters_state['brightness'])
        
        if self.filters_state['contrast'] != 0:
            img = ImageFilters.adjust_contrast(img, self.filters_state['contrast'])
        
        if self.filters_state['saturation'] != 0:
            img = ImageFilters.adjust_saturation(img, self.filters_state['saturation'])
        
        if self.filters_state['sharpness'] != 0:
            img = ImageFilters.adjust_sharpness(img, self.filters_state['sharpness'])
        
        if self.filters_state['blur'] > 0:
            img = ImageFilters.apply_blur(img, self.filters_state['blur'])
        
        self.preview_image = img
        self.update_preview()
    
    def apply_grayscale(self):
        """Aplicar filtro de escala de grises"""
        self.preview_image = ImageFilters.convert_grayscale(self.preview_image)
        self.update_preview()
    
    def apply_sepia(self):
        """Aplicar filtro sepia"""
        self.preview_image = ImageFilters.apply_sepia(self.preview_image)
        self.update_preview()
    
    def apply_invert(self):
        """Invertir colores"""
        self.preview_image = ImageFilters.invert_colors(self.preview_image)
        self.update_preview()
    
    def apply_vignette(self):
        """Aplicar viñeta"""
        self.preview_image = ImageFilters.apply_vignette(self.preview_image)
        self.update_preview()
    
    def reset_all_filters(self):
        """Restablecer todos los filtros"""
        self.brightness_slider.setValue(0)
        self.contrast_slider.setValue(0)
        self.saturation_slider.setValue(0)
        self.sharpness_slider.setValue(0)
        self.blur_slider.setValue(0)
        
        self.filters_state = {
            'brightness': 0.0,
            'contrast': 0.0,
            'saturation': 0.0,
            'sharpness': 0.0,
            'blur': 0.0,
        }
        
        self.preview_image = self.original_image.copy()
        self.update_preview()
    
    def update_preview(self):
        """Actualizar preview en el label"""
        # Redimensionar para preview
        preview_size = (400, 400)
        preview_img = self.preview_image.copy()
        preview_img.thumbnail(preview_size, Image.Resampling.LANCZOS)
        
        # Convertir a QPixmap
        if preview_img.mode == 'RGBA':
            qimg = ImageQt.ImageQt(preview_img)
        else:
            preview_img_rgb = preview_img.convert('RGB')
            data = preview_img_rgb.tobytes("raw", "RGB")
            qimg = QImage(data, preview_img_rgb.width, preview_img_rgb.height, 
                         QImage.Format.Format_RGB888)
        
        pixmap = QPixmap.fromImage(qimg)
        self.preview_label.setPixmap(pixmap)
    
    def show_original_preview(self):
        """Mostrar imagen original temporalmente"""
        preview_size = (400, 400)
        preview_img = self.original_image.copy()
        preview_img.thumbnail(preview_size, Image.Resampling.LANCZOS)
        
        if preview_img.mode == 'RGBA':
            qimg = ImageQt.ImageQt(preview_img)
        else:
            preview_img_rgb = preview_img.convert('RGB')
            data = preview_img_rgb.tobytes("raw", "RGB")
            qimg = QImage(data, preview_img_rgb.width, preview_img_rgb.height, 
                         QImage.Format.Format_RGB888)
        
        pixmap = QPixmap.fromImage(qimg)
        self.preview_label.setPixmap(pixmap)
    
    def show_filtered_preview(self):
        """Volver a mostrar imagen con filtros"""
        self.update_preview()
    
    def get_filtered_image(self):
        """Obtener imagen con filtros aplicados"""
        return self.preview_image

# Agregar botón en el panel de propiedades:
def open_image_filters(self):
    """Abrir diálogo de filtros para imagen seleccionada"""
    selected = [item for item in self.scene.selectedItems() 
                if isinstance(item, DraggableImageItem)]
    
    if not selected:
        QMessageBox.information(self, "Info", "Selecciona una imagen primero")
        return
    
    if len(selected) > 1:
        QMessageBox.information(self, "Info", 
                              "Selecciona solo una imagen para aplicar filtros")
        return
    
    image_item = selected[0]
    
    dialog = ImageFiltersDialog(image_item, self)
    if dialog.exec() == QDialog.DialogCode.Accepted:
        # Guardar imagen filtrada
        filtered_img = dialog.get_filtered_image()
        
        # Guardar en archivo temporal
        temp_path = os.path.join(tempfile.gettempdir(), 
                                f"filtered_{uuid.uuid4()}.png")
        filtered_img.save(temp_path)
        
        # Registrar en TempFileManager
        TempFileManager.get_instance().register_temp_file(temp_path)
        
        # Actualizar canvas_item
        image_item.canvas_item.image_path = temp_path
        
        # Actualizar pixmap
        width_px = cm_to_pixels(image_item.canvas_item.width, self.canvas_dpi)
        height_px = cm_to_pixels(image_item.canvas_item.height, self.canvas_dpi)
        
        if filtered_img.mode == 'RGBA':
            qimg = ImageQt.ImageQt(filtered_img)
            pixmap = QPixmap.fromImage(qimg)
        else:
            filtered_img_rgb = filtered_img.convert('RGB')
            data = filtered_img_rgb.tobytes("raw", "RGB")
            qimg = QImage(data, filtered_img_rgb.width, filtered_img_rgb.height, 
                         QImage.Format.Format_RGB888)
            pixmap = QPixmap.fromImage(qimg)
        
        scaled_pixmap = pixmap.scaled(
            int(width_px), int(height_px),
            Qt.AspectRatioMode.IgnoreAspectRatio,
            Qt.TransformationMode.SmoothTransformation
        )
        
        image_item.setPixmap(scaled_pixmap)
        
        self.save_history_state()
        self.statusBar().showMessage("Filtros aplicados correctamente", 3000)
```

---

### **2.2 EFECTOS (SOMBRA Y CONTORNO)**

Sistema de efectos aplicables a cualquier objeto.

```python
@dataclass
class EffectsSettings:
    """
    Configuración de efectos para cualquier objeto
    """
    # Sombra
    shadow_enabled: bool = False
    shadow_offset_x: float = 5.0  # en píxeles
    shadow_offset_y: float = 5.0
    shadow_blur_radius: float = 10.0
    shadow_color: str = "#00000080"  # RGBA
    
    # Contorno exterior
    outer_stroke_enabled: bool = False
    outer_stroke_width: float = 3.0
    outer_stroke_color: str = "#ffffff"
    
    # Resplandor
    glow_enabled: bool = False
    glow_radius: float = 15.0
    glow_color: str = "#00c4cc80"
    
    # Blur del objeto completo
    object_blur: float = 0.0

class EffectsRenderer:
    """
    Renderizador de efectos para objetos
    """
    
    @staticmethod
    def apply_shadow_to_pixmap(pixmap: QPixmap, settings: EffectsSettings) -> QPixmap:
        """
        Aplicar sombra a un QPixmap
        Retorna nuevo pixmap con sombra incluida
        """
        if not settings.shadow_enabled:
            return pixmap
        
        # Calcular tamaño necesario para sombra
        offset_x = int(settings.shadow_offset_x)
        offset_y = int(settings.shadow_offset_y)
        blur = int(settings.shadow_blur_radius)
        
        new_width = pixmap.width() + abs(offset_x) + blur * 2
        new_height = pixmap.height() + abs(offset_y) + blur * 2
        
        # Crear pixmap más grande
        result = QPixmap(new_width, new_height)
        result.fill(Qt.GlobalColor.transparent)
        
        painter = QPainter(result)
        painter.setRenderHint(QPainter.RenderHint.Antialiasing)
        
        # Dibujar sombra
        shadow_x = blur + max(0, offset_x)
        shadow_y = blur + max(0, offset_y)
        
        # Crear sombra
        shadow_pixmap = QPixmap(pixmap.size())
        shadow_pixmap.fill(QColor(settings.shadow_color))
        shadow_pixmap.setMask(pixmap.mask())
        
        # Aplicar blur (simplificado, idealmente usar QGraphicsBlurEffect)
        painter.setOpacity(0.5)
        for i in range(int(blur)):
            painter.drawPixmap(
                shadow_x + i - blur//2,
                shadow_y + i - blur//2,
                shadow_pixmap
            )
        
        # Dibujar imagen original
        painter.setOpacity(1.0)
        img_x = blur + max(0, -offset_x)
        img_y = blur + max(0, -offset_y)
        painter.drawPixmap(img_x, img_y, pixmap)
        
        painter.end()
        
        return result
    
    @staticmethod
    def apply_outer_stroke_to_pixmap(pixmap: QPixmap, settings: EffectsSettings) -> QPixmap:
        """
        Aplicar contorno exterior a un pixmap
        """
        if not settings.outer_stroke_enabled:
            return pixmap
        
        stroke_width = int(settings.outer_stroke_width)
        new_width = pixmap.width() + stroke_width * 2
        new_height = pixmap.height() + stroke_width * 2
        
        result = QPixmap(new_width, new_height)
        result.fill(Qt.GlobalColor.transparent)
        
        painter = QPainter(result)
        painter.setRenderHint(QPainter.RenderHint.Antialiasing)
        
        # Dibujar contorno (múltiples capas)
        stroke_color = QColor(settings.outer_stroke_color)
        for offset in range(stroke_width):
            painter.setOpacity(1.0 - (offset / stroke_width) * 0.5)
            painter.drawPixmap(
                stroke_width - offset,
                stroke_width - offset,
                pixmap
            )
            painter.drawPixmap(
                stroke_width + offset,
                stroke_width - offset,
                pixmap
            )
            painter.drawPixmap(
                stroke_width - offset,
                stroke_width + offset,
                pixmap
            )
            painter.drawPixmap(
                stroke_width + offset,
                stroke_width + offset,
                pixmap
            )
        
        # Dibujar imagen original encima
        painter.setOpacity(1.0)
        painter.drawPixmap(stroke_width, stroke_width, pixmap)
        
        painter.end()
        
        return result

# Agregar efectos a CanvasImageItem:
@dataclass
class CanvasImageItem:
    # ... campos existentes ...
    effects: EffectsSettings = field(default_factory=EffectsSettings)

# Lo mismo para TextCanvasItem y ShapeCanvasItem

# Diálogo de efectos:
class EffectsDialog(QDialog):
    """
    Diálogo para configurar efectos de objetos
    """
    
    def __init__(self, canvas_item, parent=None):
        super().__init__(parent)
        self.canvas_item = canvas_item
        self.effects = canvas_item.effects if hasattr(canvas_item, 'effects') else EffectsSettings()
        
        self.setWindowTitle("✨ Efectos")
        self.resize(400, 500)
        self.setup_ui()
    
    def setup_ui(self):
        layout = QVBoxLayout()
        
        # SOMBRA
        shadow_group = QGroupBox("🌑 Sombra")
        shadow_layout = QFormLayout()
        
        self.shadow_enabled = QCheckBox("Activar sombra")
        self.shadow_enabled.setChecked(self.effects.shadow_enabled)
        self.shadow_enabled.stateChanged.connect(self.update_preview)
        
        self.shadow_offset_x = QSpinBox()
        self.shadow_offset_x.setRange(-50, 50)
        self.shadow_offset_x.setValue(int(self.effects.shadow_offset_x))
        self.shadow_offset_x.setSuffix(" px")
        self.shadow_offset_x.valueChanged.connect(self.update_preview)
        
        self.shadow_offset_y = QSpinBox()
        self.shadow_offset_y.setRange(-50, 50)
        self.shadow_offset_y.setValue(int(self.effects.shadow_offset_y))
        self.shadow_offset_y.setSuffix(" px")
        self.shadow_offset_y.valueChanged.connect(self.update_preview)
        
        self.shadow_blur = QSpinBox()
        self.shadow_blur.setRange(0, 50)
        self.shadow_blur.setValue(int(self.effects.shadow_blur_radius))
        self.shadow_blur.setSuffix(" px")
        self.shadow_blur.valueChanged.connect(self.update_preview)
        
        self.shadow_color_btn = QPushButton("Color")
        self.shadow_color_btn.clicked.connect(self.choose_shadow_color)
        
        shadow_layout.addRow("", self.shadow_enabled)
        shadow_layout.addRow("Desplazamiento X:", self.shadow_offset_x)
        shadow_layout.addRow("Desplazamiento Y:", self.shadow_offset_y)
        shadow_layout.addRow("Desenfoque:", self.shadow_blur)
        shadow_layout.addRow("Color:", self.shadow_color_btn)
        shadow_group.setLayout(shadow_layout)
        
        # CONTORNO
        stroke_group = QGroupBox("⭕ Contorno Exterior")
        stroke_layout = QFormLayout()
        
        self.stroke_enabled = QCheckBox("Activar contorno")
        self.stroke_enabled.setChecked(self.effects.outer_stroke_enabled)
        self.stroke_enabled.stateChanged.connect(self.update_preview)
        
        self.stroke_width = QSpinBox()
        self.stroke_width.setRange(1, 20)
        self.stroke_width.setValue(int(self.effects.outer_stroke_width))
        self.stroke_width.setSuffix(" px")
        self.stroke_width.valueChanged.connect(self.update_preview)
        
        self.stroke_color_btn = QPushButton("Color")
        self.stroke_color_btn.clicked.connect(self.choose_stroke_color)
        
        stroke_layout.addRow("", self.stroke_enabled)
        stroke_layout.addRow("Grosor:", self.stroke_width)
        stroke_layout.addRow("Color:", self.stroke_color_btn)
        stroke_group.setLayout(stroke_layout)
        
        # RESPLANDOR
        glow_group = QGroupBox("✨ Resplandor")
        glow_layout = QFormLayout()
        
        self.glow_enabled = QCheckBox("Activar resplandor")
        self.glow_enabled.setChecked(self.effects.glow_enabled)
        self.glow_enabled.stateChanged.connect(self.update_preview)
        
        self.glow_radius = QSpinBox()
        self.glow_radius.setRange(5, 50)
        self.glow_radius.setValue(int(self.effects.glow_radius))
        self.glow_radius.setSuffix(" px")
        self.glow_radius.valueChanged.connect(self.update_preview)
        
        self.glow_color_btn = QPushButton("Color")
        self.glow_color_btn.clicked.connect(self.choose_glow_color)
        
        glow_layout.addRow("", self.glow_enabled)
        glow_layout.addRow("Radio:", self.glow_radius)
        glow_layout.addRow("Color:", self.glow_color_btn)
        glow_group.setLayout(glow_layout)
        
        # Preview
        self.preview_label = QLabel("Vista previa aquí")
        self.preview_label.setAlignment(Qt.AlignmentFlag.AlignCenter)
        self.preview_label.setStyleSheet("border: 1px solid #ccc; min-height: 150px;")
        
        # Botones
        buttons = QDialogButtonBox(
            QDialogButtonBox.StandardButton.Ok | 
            QDialogButtonBox.StandardButton.Cancel
        )
        buttons.accepted.connect(self.accept)
        buttons.rejected.connect(self.reject)
        
        layout.addWidget(shadow_group)
        layout.addWidget(stroke_group)
        layout.addWidget(glow_group)
        layout.addWidget(QLabel("<b>Vista Previa:</b>"))
        layout.addWidget(self.preview_label)
        layout.addWidget(buttons)
        
        self.setLayout(layout)
        self.update_preview()
    
    def update_preview(self):
        """Actualizar preview con efectos"""
        # Implementar preview simple
        pass
    
    def choose_shadow_color(self):
        color = QColorDialog.getColor(QColor(self.effects.shadow_color))
        if color.isValid():
            self.effects.shadow_color = color.name()
            self.update_preview()
    
    def choose_stroke_color(self):
        color = QColorDialog.getColor(QColor(self.effects.outer_stroke_color))
        if color.isValid():
            self.effects.outer_stroke_color = color.name()
            self.update_preview()
    
    def choose_glow_color(self):
        color = QColorDialog.getColor(QColor(self.effects.glow_color))
        if color.isValid():
            self.effects.glow_color = color.name()
            self.update_preview()
    
    def get_effects(self) -> EffectsSettings:
        """Obtener configuración de efectos"""
        effects = EffectsSettings()
        effects.shadow_enabled = self.shadow_enabled.isChecked()
        effects.shadow_offset_x = self.shadow_offset_x.value()
        effects.shadow_offset_y = self.shadow_offset_y.value()
        effects.shadow_blur_radius = self.shadow_blur.value()
        effects.shadow_color = self.effects.shadow_color
        
        effects.outer_stroke_enabled = self.stroke_enabled.isChecked()
        effects.outer_stroke_width = self.stroke_width.value()
        effects.outer_stroke_color = self.effects.outer_stroke_color
        
        effects.glow_enabled = self.glow_enabled.isChecked()
        effects.glow_radius = self.glow_radius.value()
        effects.glow_color = self.effects.glow_color
        
        return effects
```

---

### **2.3 COLOR PICKER PROFESIONAL**

Reemplazar QColorDialog básico con uno avanzado.

```python
class ProfessionalColorPicker(QDialog):
    """
    Color picker profesional con múltiples modos
    """
    
    def __init__(self, initial_color: QColor = None, parent=None):
        super().__init__(parent)
        self.current_color = initial_color or QColor(255, 0, 0)
        self.recent_colors = []
        self.saved_palettes = []
        
        self.setWindowTitle("🎨 Selector de Color")
        self.resize(600, 500)
        self.setup_ui()
    
    def setup_ui(self):
        layout = QVBoxLayout()
        
        # Tabs para diferentes selectores
        tabs = QTabWidget()
        
        # Tab 1: Rueda de color
        wheel_widget = self.create_color_wheel_tab()
        tabs.addTab(wheel_widget, "🎡 Rueda")
        
        # Tab 2: Sliders RGB/HSV
        sliders_widget = self.create_sliders_tab()
        tabs.addTab(sliders_widget, "🎚️ Sliders")
        
        # Tab 3: Paletas
        palettes_widget = self.create_palettes_tab()
        tabs.addTab(palettes_widget, "🎨 Paletas")
        
        # Preview del color actual
        preview_layout = QHBoxLayout()
        
        self.preview_label = QLabel()
        self.preview_label.setFixedSize(100, 100)
        self.preview_label.setStyleSheet(f"background: {self.current_color.name()}; border: 2px solid #000;")
        
        color_info_layout = QVBoxLayout()
        self.hex_input = QLineEdit(self.current_color.name())
        self.hex_input.textChanged.connect(self.on_hex_changed)
        
        self.rgb_label = QLabel(f"RGB: {self.current_color.red()}, {self.current_color.green()}, {self.current_color.blue()}")
        self.hsv_label = QLabel(f"HSV: {self.current_color.hue()}, {self.current_color.saturation()}, {self.current_color.value()}")
        
        color_info_layout.addWidget(QLabel("<b>HEX:</b>"))
        color_info_layout.addWidget(self.hex_input)
        color_info_layout.addWidget(self.rgb_label)
        color_info_layout.addWidget(self.hsv_label)
        
        preview_layout.addWidget(self.preview_label)
        preview_layout.addLayout(color_info_layout)
        preview_layout.addStretch()
        
        # Colores recientes
        recent_group = QGroupBox("🕐 Colores Recientes")
        recent_layout = QHBoxLayout()
        self.recent_colors_layout = recent_layout
        recent_group.setLayout(recent_layout)
        
        # Botones
        buttons = QDialogButtonBox(
            QDialogButtonBox.StandardButton.Ok | 
            QDialogButtonBox.StandardButton.Cancel
        )
        buttons.accepted.connect(self.accept)
        buttons.rejected.connect(self.reject)
        
        layout.addWidget(tabs)
        layout.addLayout(preview_layout)
        layout.addWidget(recent_group)
        layout.addWidget(buttons)
        
        self.setLayout(layout)
    
    def create_color_wheel_tab(self):
        """Crear tab con rueda de color"""
        widget = QWidget()
        layout = QVBoxLayout()
        
        # Implementar rueda de color personalizada
        # (Esto requiere QPainter y cálculos de HSV)
        
        label = QLabel("Rueda de color (implementar con QPainter)")
        label.setAlignment(Qt.AlignmentFlag.AlignCenter)
        layout.addWidget(label)
        
        widget.setLayout(layout)
        return widget
    
    def create_sliders_tab(self):
        """Crear tab con sliders RGB y HSV"""
        widget = QWidget()
        layout = QVBoxLayout()
        
        # RGB Sliders
        rgb_group = QGroupBox("RGB")
        rgb_layout = QFormLayout()
        
        self.r_slider = QSlider(Qt.Orientation.Horizontal)
        self.r_slider.setRange(0, 255)
        self.r_slider.setValue(self.current_color.red())
        self.r_slider.valueChanged.connect(self.on_rgb_changed)
        self.r_spinbox = QSpinBox()
        self.r_spinbox.setRange(0, 255)
        self.r_spinbox.setValue(self.current_color.red())
        self.r_spinbox.valueChanged.connect(self.r_slider.setValue)
        
        self.g_slider = QSlider(Qt.Orientation.Horizontal)
        self.g_slider.setRange(0, 255)
        self.g_slider.setValue(self.current_color.green())
        self.g_slider.valueChanged.connect(self.on_rgb_changed)
        self.g_spinbox = QSpinBox()
        self.g_spinbox.setRange(0, 255)
        self.g_spinbox.setValue(self.current_color.green())
        self.g_spinbox.valueChanged.connect(self.g_slider.setValue)
        
        self.b_slider = QSlider(Qt.Orientation.Horizontal)
        self.b_slider.setRange(0, 255)
        self.b_slider.setValue(self.current_color.blue())
        self.b_slider.valueChanged.connect(self.on_rgb_changed)
        self.b_spinbox = QSpinBox()
        self.b_spinbox.setRange(0, 255)
        self.b_spinbox.setValue(self.current_color.blue())
        self.b_spinbox.valueChanged.connect(self.b_slider.setValue)
        
        r_layout = QHBoxLayout()
        r_layout.addWidget(self.r_slider)
        r_layout.addWidget(self.r_spinbox)
        
        g_layout = QHBoxLayout()
        g_layout.addWidget(self.g_slider)
        g_layout.addWidget(self.g_spinbox)
        
        b_layout = QHBoxLayout()
        b_layout.addWidget(self.b_slider)
        b_layout.addWidget(self.b_spinbox)
        
        rgb_layout.addRow("🔴 Rojo:", r_layout)
        rgb_layout.addRow("🟢 Verde:", g_layout)
        rgb_layout.addRow("🔵 Azul:", b_layout)
        rgb_group.setLayout(rgb_layout)
        
        layout.addWidget(rgb_group)
        layout.addStretch()
        
        widget.setLayout(layout)
        return widget
    
    def create_palettes_tab(self):
        """Crear tab con paletas predefinidas"""
        widget = QWidget()
        layout = QVBoxLayout()
        
        # Paletas Material Design, Flat Colors, etc.
        material_colors = [
            "#F44336", "#E91E63", "#9C27B0", "#673AB7",
            "#3F51B5", "#2196F3", "#03A9F4", "#00BCD4",
            "#009688", "#4CAF50", "#8BC34A", "#CDDC39",
            "#FFEB3B", "#FFC107", "#FF9800", "#FF5722"
        ]
        
        grid = QGridLayout()
        for i, color in enumerate(material_colors):
            btn = QPushButton()
            btn.setFixedSize(40, 40)
            btn.setStyleSheet(f"background: {color}; border: 1px solid #000;")
            btn.clicked.connect(lambda checked, c=color: self.set_color_from_hex(c))
            grid.addWidget(btn, i // 8, i % 8)
        
        layout.addWidget(QLabel("<b>Material Design Colors</b>"))
        layout.addLayout(grid)
        layout.addStretch()
        
        widget.setLayout(layout)
        return widget
    
    def on_rgb_changed(self):
        """Actualizar color cuando cambian sliders RGB"""
        r = self.r_slider.value()
        g = self.g_slider.value()
        b = self.b_slider.value()
        
        self.current_color = QColor(r, g, b)
        self.update_preview()
    
    def on_hex_changed(self, text):
        """Actualizar color cuando cambia HEX"""
        if QColor.isValidColor(text):
            self.current_color = QColor(text)
            self.update_sliders_from_color()
            self.update_preview()
    
    def set_color_from_hex(self, hex_color):
        """Establecer color desde HEX"""
        self.current_color = QColor(hex_color)
        self.hex_input.setText(hex_color)
        self.update_sliders_from_color()
        self.update_preview()
    
    def update_sliders_from_color(self):
        """Actualizar sliders desde color actual"""
        self.r_slider.blockSignals(True)
        self.g_slider.blockSignals(True)
        self.b_slider.blockSignals(True)
        
        self.r_slider.setValue(self.current_color.red())
        self.g_slider.setValue(self.current_color.green())
        self.b_slider.setValue(self.current_color.blue())
        
        self.r_spinbox.setValue(self.current_color.red())
        self.g_spinbox.setValue(self.current_color.green())
        self.b_spinbox.setValue(self.current_color.blue())
        
        self.r_slider.blockSignals(False)
        self.g_slider.blockSignals(False)
        self.b_slider.blockSignals(False)
    
    def update_preview(self):
        """Actualizar preview del color"""
        self.preview_label.setStyleSheet(
            f"background: {self.current_color.name()}; border: 2px solid #000;"
        )
        self.hex_input.blockSignals(True)
        self.hex_input.setText(self.current_color.name())
        self.hex_input.blockSignals(False)
        
        self.rgb_label.setText(
            f"RGB: {self.current_color.red()}, "
            f"{self.current_color.green()}, "
            f"{self.current_color.blue()}"
        )
        self.hsv_label.setText(
            f"HSV: {self.current_color.hue()}, "
            f"{self.current_color.saturation()}, "
            f"{self.current_color.value()}"
        )
    
    def get_color(self) -> QColor:
        """Obtener color seleccionado"""
        return self.current_color
```

---

### **2.4 GUÍAS INTELIGENTES CON SNAP**

Sistema de guías que aparecen al mover objetos.

```python
class SmartGuides:
    """
    Sistema de guías inteligentes que aparecen al alinear objetos
    """
    
    def __init__(self, canvas_editor):
        self.editor = canvas_editor
        self.active_guides = []
        self.snap_threshold = 5  # píxeles
        self.guide_color = QColor(255, 0, 255)  # Fucsia como Figma
        
    def check_alignment(self, moving_item, all_items):
        """
        Verificar alineación del objeto en movimiento con otros
        Retorna posición ajustada si hay snap
        """
        self.clear_guides()
        
        moving_rect = moving_item.sceneBoundingRect()
        moving_center = moving_rect.center()
        moving_left = moving_rect.left()
        moving_right = moving_rect.right()
        moving_top = moving_rect.top()
        moving_bottom = moving_rect.bottom()
        
        snap_x = None
        snap_y = None
        
        for item in all_items:
            if item == moving_item:
                continue
            
            rect = item.sceneBoundingRect()
            center = rect.center()
            
            # Check horizontal alignment
            # Centro con centro
            if abs(moving_center.x() - center.x()) < self.snap_threshold:
                snap_x = center.x() - moving_rect.width() / 2
                self.add_vertical_guide(center.x())
            
            # Izquierda con izquierda
            elif abs(moving_left - rect.left()) < self.snap_threshold:
                snap_x = rect.left()
                self.add_vertical_guide(rect.left())
            
            # Derecha con derecha
            elif abs(moving_right - rect.right()) < self.snap_threshold:
                snap_x = rect.right() - moving_rect.width()
                self.add_vertical_guide(rect.right())
            
            # Izquierda con derecha
            elif abs(moving_left - rect.right()) < self.snap_threshold:
                snap_x = rect.right()
                self.add_vertical_guide(rect.right())
            
            # Derecha con izquierda
            elif abs(moving_right - rect.left()) < self.snap_threshold:
                snap_x = rect.left() - moving_rect.width()
                self.add_vertical_guide(rect.left())
            
            # Check vertical alignment
            # Centro con centro
            if abs(moving_center.y() - center.y()) < self.snap_threshold:
                snap_y = center.y() - moving_rect.height() / 2
                self.add_horizontal_guide(center.y())
            
            # Arriba con arriba
            elif abs(moving_top - rect.top()) < self.snap_threshold:
                snap_y = rect.top()
                self.add_horizontal_guide(rect.top())
            
            # Abajo con abajo
            elif abs(moving_bottom - rect.bottom()) < self.snap_threshold:
                snap_y = rect.bottom() - moving_rect.height()
                self.add_horizontal_guide(rect.bottom())
            
            # Arriba con abajo
            elif abs(moving_top - rect.bottom()) < self.snap_threshold:
                snap_y = rect.bottom()
                self.add_horizontal_guide(rect.bottom())
            
            # Abajo con arriba
            elif abs(moving_bottom - rect.top()) < self.snap_threshold:
                snap_y = rect.top() - moving_rect.height()
                self.add_horizontal_guide(rect.top())
        
        # Check alignment with canvas edges
        canvas_rect = self.editor.scene.sceneRect()
        canvas_center_x = canvas_rect.center().x()
        canvas_center_y = canvas_rect.center().y()
        
        if abs(moving_center.x() - canvas_center_x) < self.snap_threshold:
            snap_x = canvas_center_x - moving_rect.width() / 2
            self.add_vertical_guide(canvas_center_x)
        
        if abs(moving_center.y() - canvas_center_y) < self.snap_threshold:
            snap_y = canvas_center_y - moving_rect.height() / 2
            self.add_horizontal_guide(canvas_center_y)
        
        return snap_x, snap_y
    
    def add_vertical_guide(self, x):
        """Agregar guía vertical"""
        line = self.editor.scene.addLine(
            x, 0,
            x, self.editor.scene.sceneRect().height(),
            QPen(self.guide_color, 1, Qt.PenStyle.DashLine)
        )
        line.setZValue(10000)  # Muy arriba
        self.active_guides.append(line)
    
    def add_horizontal_guide(self, y):
        """Agregar guía horizontal"""
        line = self.editor.scene.addLine(
            0, y,
            self.editor.scene.sceneRect().width(), y,
            QPen(self.guide_color, 1, Qt.PenStyle.DashLine)
        )
        line.setZValue(10000)
        self.active_guides.append(line)
    
    def clear_guides(self):
        """Eliminar todas las guías activas"""
        for guide in self.active_guides:
            self.editor.scene.removeItem(guide)
        self.active_guides.clear()

# Modificar itemChange en DraggableImageItem (y otros items):
def itemChange(self, change, value):
    if change == QGraphicsItem.GraphicsItemChange.ItemPositionChange:
        if hasattr(self.canvas_editor, 'smart_guides'):
            all_items = [item for item in self.scene().items() 
                        if isinstance(item, (DraggableImageItem, DraggableTextItem, DraggableShapeItem))]
            
            # Obtener snap sugerido
            snap_x, snap_y = self.canvas_editor.smart_guides.check_alignment(self, all_items)
            
            new_pos = value
            if snap_x is not None:
                new_pos.setX(snap_x)
            if snap_y is not None:
                new_pos.setY(snap_y)
            
            return new_pos
    
    elif change == QGraphicsItem.GraphicsItemChange.ItemPositionHasChanged:
        # Limpiar guías cuando se suelta
        if hasattr(self.canvas_editor, 'smart_guides'):
            QTimer.singleShot(100, self.canvas_editor.smart_guides.clear_guides)
        
        # ... resto del código original
    
    return super().itemChange(change, value)

# En CanvasEditor.__init__:
self.smart_guides = SmartGuides(self)
```

---

## 📝 RESUMEN DE ENTREGABLES

### ✅ FASE 1 - FUNCIONALIDAD CORE
1. **Bugs corregidos** (7 bugs críticos)
2. **Herramienta de texto completa** con edición in-place
3. **Formas geométricas** (rectángulo, círculo, triángulo, estrella, polígono, línea)
4. **Sistema de alineación** (izq, centro, der, arriba, medio, abajo, distribuir H/V)

### ✅ FASE 2 - MEJORAS VISUALES
5. **Filtros de imagen** (brillo, contraste, saturación, nitidez, blur, B&N, sepia, invertir, viñeta)
6. **Efectos** (sombra, contorno exterior, resplandor)
7. **Color picker profesional** (RGB, HSV, HEX, paletas)
8. **Guías inteligentes** (snap automático a otros objetos y canvas)

---

## 🎯 INSTRUCCIONES FINALES PARA EL AGENTE

### **PRIORIDAD #1: CORREGIR TODOS LOS BUGS**
Antes de agregar funcionalidades nuevas, DEBES corregir los 7 bugs identificados.

### **PRIORIDAD #2: TEXTO**
La herramienta de texto es CRÍTICA. Debe funcionar perfectamente con:
- Doble click para editar
- Propiedades completas (fuente, tamaño, color, alineación)
- Efectos (sombra, contorno)

### **PRIORIDAD #3: FORMAS**
Implementar al menos: rectángulo, círculo, triángulo con propiedades de relleno y borde.

### **PRIORIDAD #4: ALINEACIÓN**
Sistema básico de alineación izq/centro/der y arriba/medio/abajo.

### **IMPLEMENTACIÓN MODULAR**
- Crea clases separadas para cada funcionalidad
- Usa dataclasses para estructuras de datos
- Implementa todo con type hints
- Documenta cada función con docstrings

### **TESTING**
Después de cada implementación, verifica que:
- No rompa funcionalidad existente
- Historial (Undo/Redo) funcione correctamente
- Exportación incluya las nuevas características

### **INTEGRACIÓN CON CÓDIGO EXISTENTE**
- Mantén la estructura actual del proyecto
- Reutiliza métodos existentes cuando sea posible
- Actualiza `restore_state()` para incluir nuevos tipos de objetos
- Actualiza `export_to_pdf()` y `export_to_image()` para nuevos objetos

---

## 🚀 COMENZAR IMPLEMENTACIÓN

**AGENTE DE GITHUB COPILOT**: 

1. Lee TODO este prompt cuidadosamente
2. Analiza el código existente en `canvas_editor.py`
3. Comienza por **FASE 1.1 - CORRECCIÓN DE BUGS**
4. Continúa con **FASE 1.2 - HERRAMIENTA DE TEXTO**
5. Luego **FASE 1.3 - FORMAS GEOMÉTRICAS**
6. Después **FASE 1.4 - ALINEACIÓN**
7. Finalmente **TODA LA FASE 2**

**NOTA IMPORTANTE**: Este prompt contiene código completo y funcional. No necesitas inventar nada, solo implementar lo especificado. Si algo no está claro, implementa la solución más robusta y profesional posible.

**TIEMPO ESTIMADO**: 
- FASE 1: 2-3 semanas de desarrollo
- FASE 2: 2-3 semanas de desarrollo
- **TOTAL**: 4-6 semanas para un desarrollo completo y profesional

**ÉXITO ESPERADO**: Al completar estas dos fases, el editor de canvas será **COMPLETO Y PROFESIONAL**, con las características esenciales para competir con Canva en funcionalidad básica.

---

## ✨ ¡ADELANTE! IMPLEMENTA TODO LO ESPECIFICADO.
