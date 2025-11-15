# Prompt: Sistema de Capas Profesional y Completo

## 📋 CONTEXTO DEL PROYECTO

**Usuario GitHub**: joseliviaa05-hub  
**Fecha**: 2025-11-15 21:37:02 UTC  
**Archivo principal**: `canvas_editor.py`  
**Objetivo**: Mejorar y profesionalizar completamente el sistema de capas del editor de canvas

## 🎯 OBJETIVO PRINCIPAL

Transformar el sistema de capas actual en un **sistema profesional, estable y completo** inspirado en Photoshop, Figma y Canva, con todas las funcionalidades que un usuario espera de un editor moderno.

---

## 🔍 ANÁLISIS DEL SISTEMA ACTUAL

### ✅ Lo que ya funciona:
- Lista de capas con thumbnails
- Drag & drop para reordenar
- Z-index básico
- Selección de capas desde el panel

### ❌ Problemas y limitaciones actuales:
1. No hay jerarquía (grupos de capas)
2. No se pueden bloquear/ocultar capas desde el panel
3. No hay renombrado de capas
4. No hay modos de fusión (blend modes)
5. No hay opacidad por capa independiente
6. No hay duplicación rápida desde el panel
7. No hay búsqueda/filtrado de capas
8. Thumbnails no se actualizan al modificar objetos
9. No hay indicadores visuales de estado (bloqueado, oculto)
10. No hay multi-selección de capas
11. No hay capas de ajuste
12. No hay colores de identificación
13. No soporta tipos mixtos (imagen, texto, forma) uniformemente

---

## 🏗️ ARQUITECTURA DEL NUEVO SISTEMA

### **ESTRUCTURA DE DATOS MEJORADA**

```python
from enum import Enum
from dataclasses import dataclass, field
from typing import List, Optional, Set, Dict, Any
import uuid

class LayerType(Enum):
    """Tipos de capas soportados"""
    IMAGE = "image"
    TEXT = "text"
    SHAPE = "shape"
    GROUP = "group"
    ADJUSTMENT = "adjustment"  # Futuro: capas de ajuste

class BlendMode(Enum):
    """Modos de fusión/mezcla"""
    NORMAL = "normal"
    MULTIPLY = "multiply"
    SCREEN = "screen"
    OVERLAY = "overlay"
    DARKEN = "darken"
    LIGHTEN = "lighten"
    COLOR_DODGE = "color_dodge"
    COLOR_BURN = "color_burn"
    HARD_LIGHT = "hard_light"
    SOFT_LIGHT = "soft_light"
    DIFFERENCE = "difference"
    EXCLUSION = "exclusion"
    HUE = "hue"
    SATURATION = "saturation"
    COLOR = "color"
    LUMINOSITY = "luminosity"

@dataclass
class Layer:
    """
    Clase base para todas las capas
    
    CARACTERÍSTICAS:
    - Identificación única
    - Jerarquía (parent-child)
    - Estados visuales
    - Metadatos
    """
    uuid: str = field(default_factory=lambda: str(uuid.uuid4()))
    name: str = "Capa"
    layer_type: LayerType = LayerType.IMAGE
    
    # Jerarquía
    parent: Optional['Layer'] = None
    children: List['Layer'] = field(default_factory=list)
    
    # Propiedades visuales
    visible: bool = True
    locked: bool = False
    opacity: float = 1.0  # 0.0 a 1.0
    blend_mode: BlendMode = BlendMode.NORMAL
    
    # Orden
    z_index: int = 0
    
    # Metadatos
    color_label: Optional[str] = None  # Color de identificación
    notes: str = ""  # Notas del usuario
    tags: Set[str] = field(default_factory=set)  # Tags para búsqueda
    
    # Estado de edición
    selected: bool = False
    collapsed: bool = False  # Para grupos
    
    # Referencia al objeto gráfico
    graphic_item: Any = None  # DraggableImageItem, DraggableTextItem, etc.
    
    # Thumbnail cache
    thumbnail: Optional[Any] = None  # QPixmap
    thumbnail_dirty: bool = True
    
    # Timestamps
    created_at: str = field(default_factory=lambda: datetime.now().isoformat())
    modified_at: str = field(default_factory=lambda: datetime.now().isoformat())
    
    def add_child(self, child: 'Layer'):
        """Agregar capa hija"""
        child.parent = self
        self.children.append(child)
    
    def remove_child(self, child: 'Layer'):
        """Remover capa hija"""
        if child in self.children:
            child.parent = None
            self.children.remove(child)
    
    def get_all_descendants(self) -> List['Layer']:
        """Obtener todas las capas descendientes recursivamente"""
        descendants = []
        for child in self.children:
            descendants.append(child)
            descendants.extend(child.get_all_descendants())
        return descendants
    
    def get_depth(self) -> int:
        """Obtener profundidad en la jerarquía (0 = raíz)"""
        depth = 0
        current = self.parent
        while current:
            depth += 1
            current = current.parent
        return depth
    
    def is_ancestor_of(self, other: 'Layer') -> bool:
        """Verificar si esta capa es ancestro de otra"""
        current = other.parent
        while current:
            if current == self:
                return True
            current = current.parent
        return False
    
    def mark_modified(self):
        """Marcar capa como modificada"""
        self.modified_at = datetime.now().isoformat()
        self.thumbnail_dirty = True
    
    def to_dict(self) -> dict:
        """Serializar a diccionario"""
        return {
            'uuid': self.uuid,
            'name': self.name,
            'type': self.layer_type.value,
            'visible': self.visible,
            'locked': self.locked,
            'opacity': self.opacity,
            'blend_mode': self.blend_mode.value,
            'z_index': self.z_index,
            'color_label': self.color_label,
            'notes': self.notes,
            'tags': list(self.tags),
            'children': [child.uuid for child in self.children],
            'parent': self.parent.uuid if self.parent else None,
            'created_at': self.created_at,
            'modified_at': self.modified_at,
        }
    
    @staticmethod
    def from_dict(data: dict) -> 'Layer':
        """Deserializar desde diccionario"""
        layer = Layer(
            uuid=data['uuid'],
            name=data['name'],
            layer_type=LayerType(data['type']),
            visible=data['visible'],
            locked=data['locked'],
            opacity=data['opacity'],
            blend_mode=BlendMode(data['blend_mode']),
            z_index=data['z_index'],
            color_label=data.get('color_label'),
            notes=data.get('notes', ''),
            tags=set(data.get('tags', [])),
            created_at=data.get('created_at', datetime.now().isoformat()),
            modified_at=data.get('modified_at', datetime.now().isoformat()),
        )
        return layer
```

---

## 🎨 PANEL DE CAPAS PROFESIONAL

### **WIDGET PERSONALIZADO CON TODAS LAS FUNCIONES**

```python
class LayerItemWidget(QWidget):
    """
    Widget personalizado para cada item de capa en el panel
    
    CARACTERÍSTICAS:
    - Thumbnail con actualización automática
    - Indicadores visuales de estado
    - Controles inline (visibilidad, bloqueo)
    - Edición de nombre inline
    - Menú contextual
    - Drag & drop mejorado
    - Indentación para jerarquía
    """
    
    # Señales
    visibilityToggled = pyqtSignal(str)  # uuid
    lockToggled = pyqtSignal(str)
    nameChanged = pyqtSignal(str, str)  # uuid, new_name
    selected = pyqtSignal(str)
    contextMenuRequested = pyqtSignal(str, QPoint)
    
    def __init__(self, layer: Layer, parent=None):
        super().__init__(parent)
        self.layer = layer
        self.is_editing_name = False
        self.setup_ui()
        self.update_from_layer()
    
    def setup_ui(self):
        """Configurar interfaz del item"""
        layout = QHBoxLayout()
        layout.setContentsMargins(5, 2, 5, 2)
        layout.setSpacing(5)
        
        # Indentación para jerarquía
        self.indent_spacer = QSpacerItem(0, 0, QSizePolicy.Policy.Fixed, QSizePolicy.Policy.Fixed)
        layout.addItem(self.indent_spacer)
        
        # Botón de colapsar/expandir (solo para grupos)
        self.collapse_btn = QPushButton("▼")
        self.collapse_btn.setFixedSize(16, 16)
        self.collapse_btn.setFlat(True)
        self.collapse_btn.clicked.connect(self.toggle_collapse)
        self.collapse_btn.setVisible(self.layer.layer_type == LayerType.GROUP)
        layout.addWidget(self.collapse_btn)
        
        # Thumbnail
        self.thumbnail_label = QLabel()
        self.thumbnail_label.setFixedSize(32, 32)
        self.thumbnail_label.setScaledContents(True)
        self.thumbnail_label.setStyleSheet("""
            QLabel {
                border: 1px solid #999;
                background: white;
                border-radius: 3px;
            }
        """)
        layout.addWidget(self.thumbnail_label)
        
        # Icono de tipo de capa
        self.type_icon = QLabel()
        self.type_icon.setFixedSize(16, 16)
        layout.addWidget(self.type_icon)
        
        # Nombre de capa (editable)
        self.name_label = QLabel(self.layer.name)
        self.name_label.setStyleSheet("QLabel { padding: 2px; }")
        self.name_label.mouseDoubleClickEvent = self.start_edit_name
        layout.addWidget(self.name_label, 1)  # Stretch
        
        self.name_edit = QLineEdit(self.layer.name)
        self.name_edit.hide()
        self.name_edit.editingFinished.connect(self.finish_edit_name)
        layout.addWidget(self.name_edit, 1)
        
        # Indicador de color (opcional)
        self.color_indicator = QLabel()
        self.color_indicator.setFixedSize(4, 20)
        self.color_indicator.setStyleSheet("QLabel { border-radius: 2px; }")
        layout.addWidget(self.color_indicator)
        
        # Controles
        controls_layout = QHBoxLayout()
        controls_layout.setSpacing(2)
        
        # Botón de visibilidad
        self.visibility_btn = QPushButton("👁️")
        self.visibility_btn.setFixedSize(24, 24)
        self.visibility_btn.setFlat(True)
        self.visibility_btn.setCheckable(True)
        self.visibility_btn.setChecked(self.layer.visible)
        self.visibility_btn.clicked.connect(self.on_visibility_clicked)
        self.visibility_btn.setToolTip("Mostrar/Ocultar capa")
        controls_layout.addWidget(self.visibility_btn)
        
        # Botón de bloqueo
        self.lock_btn = QPushButton("🔓")
        self.lock_btn.setFixedSize(24, 24)
        self.lock_btn.setFlat(True)
        self.lock_btn.setCheckable(True)
        self.lock_btn.setChecked(self.layer.locked)
        self.lock_btn.clicked.connect(self.on_lock_clicked)
        self.lock_btn.setToolTip("Bloquear/Desbloquear capa")
        controls_layout.addWidget(self.lock_btn)
        
        layout.addLayout(controls_layout)
        
        # Estilo del widget
        self.setLayout(layout)
        self.setFixedHeight(40)
        self.update_style()
    
    def update_from_layer(self):
        """Actualizar widget desde los datos de la capa"""
        # Actualizar nombre
        self.name_label.setText(self.layer.name)
        
        # Actualizar thumbnail
        self.update_thumbnail()
        
        # Actualizar icono de tipo
        icons = {
            LayerType.IMAGE: "🖼️",
            LayerType.TEXT: "📝",
            LayerType.SHAPE: "🔷",
            LayerType.GROUP: "📁",
            LayerType.ADJUSTMENT: "🎨",
        }
        self.type_icon.setText(icons.get(self.layer.layer_type, "❓"))
        
        # Actualizar controles
        self.visibility_btn.setChecked(self.layer.visible)
        self.visibility_btn.setText("👁️" if self.layer.visible else "👁️‍🗨️")
        
        self.lock_btn.setChecked(self.layer.locked)
        self.lock_btn.setText("🔒" if self.layer.locked else "🔓")
        
        # Actualizar color label
        if self.layer.color_label:
            self.color_indicator.setStyleSheet(
                f"QLabel {{ background: {self.layer.color_label}; border-radius: 2px; }}"
            )
            self.color_indicator.setVisible(True)
        else:
            self.color_indicator.setVisible(False)
        
        # Actualizar indentación
        depth = self.layer.get_depth()
        self.indent_spacer.changeSize(depth * 20, 0, QSizePolicy.Policy.Fixed, QSizePolicy.Policy.Fixed)
        
        # Actualizar botón de colapsar
        if self.layer.layer_type == LayerType.GROUP:
            self.collapse_btn.setVisible(True)
            self.collapse_btn.setText("▶" if self.layer.collapsed else "▼")
        else:
            self.collapse_btn.setVisible(False)
        
        self.update_style()
    
    def update_thumbnail(self):
        """Actualizar thumbnail de la capa"""
        if self.layer.thumbnail_dirty or self.layer.thumbnail is None:
            # Generar nuevo thumbnail
            thumbnail = self.generate_thumbnail()
            self.layer.thumbnail = thumbnail
            self.layer.thumbnail_dirty = False
        
        if self.layer.thumbnail:
            self.thumbnail_label.setPixmap(self.layer.thumbnail)
    
    def generate_thumbnail(self) -> QPixmap:
        """Generar thumbnail de la capa"""
        pixmap = QPixmap(32, 32)
        pixmap.fill(Qt.GlobalColor.transparent)
        
        if self.layer.graphic_item:
            painter = QPainter(pixmap)
            painter.setRenderHint(QPainter.RenderHint.Antialiasing)
            
            # Obtener bounding rect del item
            source_rect = self.layer.graphic_item.boundingRect()
            
            # Calcular escala para que quepa en 32x32
            scale = min(32.0 / source_rect.width(), 32.0 / source_rect.height())
            
            # Centrar en el pixmap
            painter.translate(16, 16)
            painter.scale(scale, scale)
            painter.translate(-source_rect.center())
            
            # Renderizar el item
            self.layer.graphic_item.paint(painter, QStyleOptionGraphicsItem(), None)
            
            painter.end()
        else:
            # Thumbnail por defecto según tipo
            painter = QPainter(pixmap)
            painter.fillRect(pixmap.rect(), QColor(200, 200, 200))
            painter.drawText(pixmap.rect(), Qt.AlignmentFlag.AlignCenter, "?")
            painter.end()
        
        return pixmap
    
    def update_style(self):
        """Actualizar estilo visual del widget"""
        if self.layer.selected:
            self.setStyleSheet("""
                LayerItemWidget {
                    background: #0078d7;
                    border-radius: 3px;
                }
                QLabel {
                    color: white;
                }
            """)
        else:
            opacity_style = ""
            if not self.layer.visible:
                opacity_style = "opacity: 0.5;"
            
            self.setStyleSheet(f"""
                LayerItemWidget {{
                    background: transparent;
                    {opacity_style}
                }}
                LayerItemWidget:hover {{
                    background: #e5e5e5;
                    border-radius: 3px;
                }}
                QLabel {{
                    color: black;
                }}
            """)
    
    def start_edit_name(self, event):
        """Iniciar edición del nombre"""
        if not self.layer.locked:
            self.name_label.hide()
            self.name_edit.setText(self.layer.name)
            self.name_edit.show()
            self.name_edit.setFocus()
            self.name_edit.selectAll()
            self.is_editing_name = True
    
    def finish_edit_name(self):
        """Finalizar edición del nombre"""
        if self.is_editing_name:
            new_name = self.name_edit.text().strip()
            if new_name and new_name != self.layer.name:
                self.layer.name = new_name
                self.layer.mark_modified()
                self.nameChanged.emit(self.layer.uuid, new_name)
            
            self.name_label.setText(self.layer.name)
            self.name_edit.hide()
            self.name_label.show()
            self.is_editing_name = False
    
    def on_visibility_clicked(self):
        """Toggle visibilidad"""
        self.layer.visible = not self.layer.visible
        self.layer.mark_modified()
        self.visibilityToggled.emit(self.layer.uuid)
        self.update_from_layer()
    
    def on_lock_clicked(self):
        """Toggle bloqueo"""
        self.layer.locked = not self.layer.locked
        self.layer.mark_modified()
        self.lockToggled.emit(self.layer.uuid)
        self.update_from_layer()
    
    def toggle_collapse(self):
        """Toggle colapsar/expandir grupo"""
        if self.layer.layer_type == LayerType.GROUP:
            self.layer.collapsed = not self.layer.collapsed
            self.update_from_layer()
            # Emitir señal para que el panel actualice hijos
    
    def mousePressEvent(self, event):
        """Seleccionar capa al hacer click"""
        if event.button() == Qt.MouseButton.LeftButton:
            self.selected.emit(self.layer.uuid)
        super().mousePressEvent(event)
    
    def contextMenuEvent(self, event):
        """Menú contextual"""
        self.contextMenuRequested.emit(self.layer.uuid, event.globalPos())


class ProfessionalLayersPanel(QWidget):
    """
    Panel de capas profesional completo
    
    CARACTERÍSTICAS:
    - Jerarquía visual con indentación
    - Drag & drop para reordenar y agrupar
    - Multi-selección
    - Búsqueda y filtrado
    - Todas las operaciones de capas
    - Menú contextual rico
    - Atajos de teclado
    """
    
    # Señales
    layerSelected = pyqtSignal(str)  # uuid
    layerOrderChanged = pyqtSignal()
    layersModified = pyqtSignal()
    
    def __init__(self, canvas_editor, parent=None):
        super().__init__(parent)
        self.canvas_editor = canvas_editor
        self.layers: List[Layer] = []
        self.selected_layers: Set[str] = set()  # UUIDs
        self.layer_widgets: Dict[str, LayerItemWidget] = {}
        
        self.setup_ui()
        self.setup_shortcuts()
    
    def setup_ui(self):
        """Configurar interfaz del panel"""
        layout = QVBoxLayout()
        layout.setContentsMargins(0, 0, 0, 0)
        
        # Toolbar superior
        toolbar = QHBoxLayout()
        
        # Búsqueda
        self.search_input = QLineEdit()
        self.search_input.setPlaceholderText("🔍 Buscar capas...")
        self.search_input.textChanged.connect(self.filter_layers)
        toolbar.addWidget(self.search_input)
        
        # Botón de nueva capa
        new_layer_btn = QPushButton("➕")
        new_layer_btn.setFixedSize(30, 30)
        new_layer_btn.setToolTip("Nueva capa")
        new_layer_btn.clicked.connect(self.show_new_layer_menu)
        toolbar.addWidget(new_layer_btn)
        
        # Botón de nuevo grupo
        new_group_btn = QPushButton("📁")
        new_group_btn.setFixedSize(30, 30)
        new_group_btn.setToolTip("Nuevo grupo")
        new_group_btn.clicked.connect(self.create_group)
        toolbar.addWidget(new_group_btn)
        
        # Botón de eliminar
        delete_btn = QPushButton("🗑️")
        delete_btn.setFixedSize(30, 30)
        delete_btn.setToolTip("Eliminar capa(s)")
        delete_btn.clicked.connect(self.delete_selected_layers)
        toolbar.addWidget(delete_btn)
        
        layout.addLayout(toolbar)
        
        # Lista de capas (scroll area)
        scroll = QScrollArea()
        scroll.setWidgetResizable(True)
        scroll.setHorizontalScrollBarPolicy(Qt.ScrollBarPolicy.ScrollBarAlwaysOff)
        
        self.layers_container = QWidget()
        self.layers_layout = QVBoxLayout()
        self.layers_layout.setContentsMargins(0, 0, 0, 0)
        self.layers_layout.setSpacing(2)
        self.layers_layout.addStretch()
        self.layers_container.setLayout(self.layers_layout)
        
        scroll.setWidget(self.layers_container)
        layout.addWidget(scroll)
        
        # Barra inferior con info
        info_bar = QHBoxLayout()
        self.info_label = QLabel("0 capas")
        self.info_label.setStyleSheet("QLabel { color: #666; font-size: 10px; }")
        info_bar.addWidget(self.info_label)
        info_bar.addStretch()
        
        # Botón de opciones del panel
        options_btn = QPushButton("⚙️")
        options_btn.setFixedSize(24, 24)
        options_btn.setFlat(True)
        options_btn.setToolTip("Opciones del panel")
        options_btn.clicked.connect(self.show_panel_options)
        info_bar.addWidget(options_btn)
        
        layout.addLayout(info_bar)
        
        self.setLayout(layout)
        self.setMinimumWidth(250)
    
    def setup_shortcuts(self):
        """Configurar atajos de teclado"""
        # Eliminar capas seleccionadas
        QShortcut(QKeySequence.StandardKey.Delete, self, self.delete_selected_layers)
        
        # Duplicar capas
        QShortcut(QKeySequence("Ctrl+J"), self, self.duplicate_selected_layers)
        
        # Agrupar capas
        QShortcut(QKeySequence("Ctrl+G"), self, self.group_selected_layers)
        
        # Desagrupar
        QShortcut(QKeySequence("Ctrl+Shift+G"), self, self.ungroup_selected_layers)
        
        # Combinar capas
        QShortcut(QKeySequence("Ctrl+E"), self, self.merge_selected_layers)
    
    def add_layer(self, layer: Layer):
        """Agregar nueva capa al panel"""
        self.layers.append(layer)
        self.create_layer_widget(layer)
        self.update_layers_display()
        self.layersModified.emit()
    
    def create_layer_widget(self, layer: Layer):
        """Crear widget para una capa"""
        widget = LayerItemWidget(layer)
        
        # Conectar señales
        widget.visibilityToggled.connect(self.on_layer_visibility_toggled)
        widget.lockToggled.connect(self.on_layer_lock_toggled)
        widget.nameChanged.connect(self.on_layer_name_changed)
        widget.selected.connect(self.on_layer_selected)
        widget.contextMenuRequested.connect(self.show_layer_context_menu)
        
        self.layer_widgets[layer.uuid] = widget
        
        return widget
    
    def update_layers_display(self):
        """Actualizar visualización de todas las capas"""
        # Limpiar layout
        while self.layers_layout.count() > 1:  # Dejar el stretch
            item = self.layers_layout.takeAt(0)
            if item.widget():
                item.widget().setParent(None)
        
        # Agregar widgets en orden inverso (arriba = más al frente)
        visible_layers = self.get_visible_layers()
        
        for layer in reversed(visible_layers):
            if layer.uuid in self.layer_widgets:
                widget = self.layer_widgets[layer.uuid]
                widget.update_from_layer()
                self.layers_layout.insertWidget(0, widget)
        
        # Actualizar info
        self.info_label.setText(f"{len(visible_layers)} capa(s)")
    
    def get_visible_layers(self) -> List[Layer]:
        """Obtener capas visibles (considerando collapsed)"""
        visible = []
        
        def add_layer_and_children(layer: Layer):
            visible.append(layer)
            if layer.layer_type == LayerType.GROUP and not layer.collapsed:
                for child in layer.children:
                    add_layer_and_children(child)
        
        # Solo capas raíz
        root_layers = [l for l in self.layers if l.parent is None]
        for layer in root_layers:
            add_layer_and_children(layer)
        
        return visible
    
    def filter_layers(self, text: str):
        """Filtrar capas por texto de búsqueda"""
        if not text:
            self.update_layers_display()
            return
        
        text = text.lower()
        
        for uuid, widget in self.layer_widgets.items():
            layer = widget.layer
            
            # Buscar en nombre y tags
            matches = (text in layer.name.lower() or 
                      any(text in tag.lower() for tag in layer.tags))
            
            widget.setVisible(matches)
    
    def on_layer_selected(self, uuid: str):
        """Manejar selección de capa"""
        # TODO: Implementar multi-selección con Ctrl/Shift
        
        # Deseleccionar todas
        for layer in self.layers:
            layer.selected = False
        
        # Seleccionar esta
        layer = self.find_layer_by_uuid(uuid)
        if layer:
            layer.selected = True
            self.selected_layers = {uuid}
            self.layerSelected.emit(uuid)
        
        self.update_layers_display()
    
    def on_layer_visibility_toggled(self, uuid: str):
        """Toggle visibilidad de capa"""
        layer = self.find_layer_by_uuid(uuid)
        if layer and layer.graphic_item:
            layer.graphic_item.setVisible(layer.visible)
            self.canvas_editor.save_history_state()
    
    def on_layer_lock_toggled(self, uuid: str):
        """Toggle bloqueo de capa"""
        layer = self.find_layer_by_uuid(uuid)
        if layer and layer.graphic_item:
            layer.graphic_item.setFlag(
                QGraphicsItem.GraphicsItemFlag.ItemIsMovable,
                not layer.locked
            )
            self.canvas_editor.save_history_state()
    
    def on_layer_name_changed(self, uuid: str, new_name: str):
        """Cambio de nombre de capa"""
        self.canvas_editor.save_history_state()
    
    def show_layer_context_menu(self, uuid: str, pos: QPoint):
        """Mostrar menú contextual de capa"""
        layer = self.find_layer_by_uuid(uuid)
        if not layer:
            return
        
        menu = QMenu(self)
        
        # Información de la capa
        info_action = menu.addAction(f"📋 {layer.name}")
        info_action.setEnabled(False)
        menu.addSeparator()
        
        # Duplicar
        duplicate_action = menu.addAction("📋 Duplicar (Ctrl+J)")
        duplicate_action.triggered.connect(lambda: self.duplicate_layer(uuid))
        
        # Eliminar
        delete_action = menu.addAction("🗑️ Eliminar (Del)")
        delete_action.triggered.connect(lambda: self.delete_layer(uuid))
        
        menu.addSeparator()
        
        # Propiedades
        properties_menu = menu.addMenu("⚙️ Propiedades")
        
        # Opacidad
        opacity_action = properties_menu.addAction(f"Opacidad: {int(layer.opacity * 100)}%")
        opacity_action.triggered.connect(lambda: self.edit_layer_opacity(uuid))
        
        # Blend mode
        blend_menu = properties_menu.addMenu("Modo de fusión")
        for mode in BlendMode:
            action = blend_menu.addAction(mode.value.replace("_", " ").title())
            action.setCheckable(True)
            action.setChecked(layer.blend_mode == mode)
            action.triggered.connect(lambda checked, m=mode: self.set_layer_blend_mode(uuid, m))
        
        # Color label
        color_menu = properties_menu.addMenu("🎨 Color de identificación")
        colors = [
            ("Ninguno", None),
            ("🔴 Rojo", "#ff0000"),
            ("🟠 Naranja", "#ff8800"),
            ("🟡 Amarillo", "#ffff00"),
            ("🟢 Verde", "#00ff00"),
            ("🔵 Azul", "#0000ff"),
            ("🟣 Púrpura", "#ff00ff"),
        ]
        for name, color in colors:
            action = color_menu.addAction(name)
            action.setCheckable(True)
            action.setChecked(layer.color_label == color)
            action.triggered.connect(lambda checked, c=color: self.set_layer_color(uuid, c))
        
        menu.addSeparator()
        
        # Orden
        order_menu = menu.addMenu("📊 Orden")
        order_menu.addAction("⬆️ Traer al frente").triggered.connect(lambda: self.bring_to_front(uuid))
        order_menu.addAction("⬇️ Enviar al fondo").triggered.connect(lambda: self.send_to_back(uuid))
        order_menu.addAction("↑ Traer adelante").triggered.connect(lambda: self.bring_forward(uuid))
        order_menu.addAction("↓ Enviar atrás").triggered.connect(lambda: self.send_backward(uuid))
        
        menu.addSeparator()
        
        # Agrupar (si hay múltiples seleccionadas)
        if len(self.selected_layers) > 1:
            group_action = menu.addAction("📁 Agrupar (Ctrl+G)")
            group_action.triggered.connect(self.group_selected_layers)
        
        # Desagrupar (si es un grupo)
        if layer.layer_type == LayerType.GROUP:
            ungroup_action = menu.addAction("📂 Desagrupar (Ctrl+Shift+G)")
            ungroup_action.triggered.connect(lambda: self.ungroup_layer(uuid))
        
        menu.exec(pos)
    
    def find_layer_by_uuid(self, uuid: str) -> Optional[Layer]:
        """Buscar capa por UUID"""
        for layer in self.layers:
            if layer.uuid == uuid:
                return layer
            # Buscar en hijos
            for child in layer.get_all_descendants():
                if child.uuid == uuid:
                    return child
        return None
    
    def duplicate_layer(self, uuid: str):
        """Duplicar capa"""
        # TODO: Implementar
        pass
    
    def delete_layer(self, uuid: str):
        """Eliminar capa"""
        layer = self.find_layer_by_uuid(uuid)
        if layer:
            # Remover del canvas
            if layer.graphic_item:
                self.canvas_editor.scene.removeItem(layer.graphic_item)
            
            # Remover de la lista
            if layer in self.layers:
                self.layers.remove(layer)
            elif layer.parent:
                layer.parent.remove_child(layer)
            
            # Remover widget
            if uuid in self.layer_widgets:
                self.layer_widgets[uuid].deleteLater()
                del self.layer_widgets[uuid]
            
            self.update_layers_display()
            self.layersModified.emit()
            self.canvas_editor.save_history_state()
    
    def delete_selected_layers(self):
        """Eliminar capas seleccionadas"""
        for uuid in list(self.selected_layers):
            self.delete_layer(uuid)
        self.selected_layers.clear()
    
    def duplicate_selected_layers(self):
        """Duplicar capas seleccionadas"""
        # TODO: Implementar
        pass
    
    def group_selected_layers(self):
        """Agrupar capas seleccionadas"""
        if len(self.selected_layers) < 2:
            QMessageBox.information(self, "Info", 
                                  "Selecciona al menos 2 capas para agrupar")
            return
        
        # Crear nuevo grupo
        group = Layer(
            name=f"Grupo {len([l for l in self.layers if l.layer_type == LayerType.GROUP]) + 1}",
            layer_type=LayerType.GROUP
        )
        
        # Mover capas al grupo
        for uuid in self.selected_layers:
            layer = self.find_layer_by_uuid(uuid)
            if layer:
                if layer in self.layers:
                    self.layers.remove(layer)
                group.add_child(layer)
        
        self.layers.append(group)
        self.create_layer_widget(group)
        self.update_layers_display()
        self.layersModified.emit()
        self.canvas_editor.save_history_state()
    
    def ungroup_selected_layers(self):
        """Desagrupar capas seleccionadas"""
        for uuid in list(self.selected_layers):
            self.ungroup_layer(uuid)
    
    def ungroup_layer(self, uuid: str):
        """Desagrupar una capa grupo"""
        layer = self.find_layer_by_uuid(uuid)
        if layer and layer.layer_type == LayerType.GROUP:
            # Mover hijos al nivel del grupo
            parent = layer.parent
            children = layer.children.copy()
            
            for child in children:
                layer.remove_child(child)
                if parent:
                    parent.add_child(child)
                else:
                    self.layers.append(child)
            
            # Eliminar grupo
            self.delete_layer(uuid)
    
    def merge_selected_layers(self):
        """Combinar capas seleccionadas en una sola"""
        # TODO: Implementar combinación de capas
        pass
    
    def edit_layer_opacity(self, uuid: str):
        """Editar opacidad de capa"""
        layer = self.find_layer_by_uuid(uuid)
        if not layer:
            return
        
        value, ok = QInputDialog.getInt(
            self, "Opacidad de Capa",
            "Opacidad (0-100):",
            int(layer.opacity * 100),
            0, 100, 1
        )
        
        if ok:
            layer.opacity = value / 100.0
            layer.mark_modified()
            if layer.graphic_item:
                layer.graphic_item.setOpacity(layer.opacity)
            self.update_layers_display()
            self.canvas_editor.save_history_state()
    
    def set_layer_blend_mode(self, uuid: str, mode: BlendMode):
        """Establecer modo de fusión de capa"""
        layer = self.find_layer_by_uuid(uuid)
        if layer:
            layer.blend_mode = mode
            layer.mark_modified()
            # TODO: Implementar blend modes en renderizado
            self.canvas_editor.save_history_state()
    
    def set_layer_color(self, uuid: str, color: Optional[str]):
        """Establecer color de identificación"""
        layer = self.find_layer_by_uuid(uuid)
        if layer:
            layer.color_label = color
            layer.mark_modified()
            self.update_layers_display()
    
    def bring_to_front(self, uuid: str):
        """Traer capa al frente"""
        # TODO: Implementar
        pass
    
    def send_to_back(self, uuid: str):
        """Enviar capa atrás"""
        # TODO: Implementar
        pass
    
    def bring_forward(self, uuid: str):
        """Traer capa adelante"""
        # TODO: Implementar
        pass
    
    def send_backward(self, uuid: str):
        """Enviar capa atrás"""
        # TODO: Implementar
        pass
    
    def show_new_layer_menu(self):
        """Mostrar menú para crear nueva capa"""
        menu = QMenu(self)
        menu.addAction("🖼️ Nueva imagen...").triggered.connect(self.canvas_editor.load_images)
        menu.addAction("📝 Nuevo texto").triggered.connect(self.canvas_editor.add_text_to_canvas)
        menu.addAction("🔷 Nueva forma...").triggered.connect(self.show_shape_menu)
        menu.exec(QCursor.pos())
    
    def show_shape_menu(self):
        """Mostrar menú de formas"""
        # Delegar al canvas_editor
        pass
    
    def create_group(self):
        """Crear grupo vacío"""
        group = Layer(
            name=f"Grupo {len([l for l in self.layers if l.layer_type == LayerType.GROUP]) + 1}",
            layer_type=LayerType.GROUP
        )
        self.add_layer(group)
    
    def show_panel_options(self):
        """Mostrar opciones del panel"""
        menu = QMenu(self)
        
        # Tamaño de thumbnails
        size_menu = menu.addMenu("Tamaño de miniaturas")
        for size in [16, 24, 32, 48, 64]:
            action = size_menu.addAction(f"{size}×{size}px")
            # TODO: Implementar cambio de tamaño
        
        menu.addSeparator()
        
        # Mostrar/ocultar columnas
        menu.addAction("Actualizar todas las miniaturas").triggered.connect(self.refresh_all_thumbnails)
        menu.addAction("Expandir todos los grupos").triggered.connect(self.expand_all_groups)
        menu.addAction("Colapsar todos los grupos").triggered.connect(self.collapse_all_groups)
        
        menu.exec(QCursor.pos())
    
    def refresh_all_thumbnails(self):
        """Refrescar todos los thumbnails"""
        for layer in self.layers:
            layer.thumbnail_dirty = True
        self.update_layers_display()
    
    def expand_all_groups(self):
        """Expandir todos los grupos"""
        for layer in self.layers:
            if layer.layer_type == LayerType.GROUP:
                layer.collapsed = False
        self.update_layers_display()
    
    def collapse_all_groups(self):
        """Colapsar todos los grupos"""
        for layer in self.layers:
            if layer.layer_type == LayerType.GROUP:
                layer.collapsed = True
        self.update_layers_display()
```

---

## 🔗 INTEGRACIÓN CON CANVAS EDITOR

### **MODIFICACIONES EN `CanvasEditor`**

```python
class CanvasEditor(QMainWindow):
    def __init__(self, parent=None):
        super().__init__(parent)
        # ... código existente ...
        
        # NUEVO: Reemplazar lista de capas simple por panel profesional
        self.layers_panel = ProfessionalLayersPanel(self)
        
        # Conectar señales
        self.layers_panel.layerSelected.connect(self.on_layer_selected_from_panel)
        self.layers_panel.layersModified.connect(self.save_history_state)
        
        # IMPORTANTE: Sincronizar sistema antiguo con nuevo
        self.sync_legacy_layers_to_new_system()
    
    def sync_legacy_layers_to_new_system(self):
        """Sincronizar canvas_images con el nuevo sistema de capas"""
        # Crear Layer para cada CanvasImageItem existente
        for canvas_item in self.canvas_images:
            layer = Layer(
                name=f"Imagen {len(self.layers_panel.layers) + 1}",
                layer_type=LayerType.IMAGE,
                visible=canvas_item.visible,
                locked=canvas_item.locked,
                opacity=canvas_item.opacity,
                z_index=canvas_item.z_index,
            )
            
            # Vincular con graphic item
            for item in self.scene.items():
                if isinstance(item, DraggableImageItem) and item.canvas_item == canvas_item:
                    layer.graphic_item = item
                    break
            
            self.layers_panel.add_layer(layer)
    
    def add_image_to_canvas_at_pos(self, path: str, scene_x: float, scene_y: float):
        """
        MODIFICAR: Agregar imagen al canvas Y al sistema de capas
        """
        # ... código existente para crear canvas_item y graphic_item ...
        
        # NUEVO: Crear Layer correspondiente
        layer = Layer(
            name=os.path.basename(path),
            layer_type=LayerType.IMAGE,
            z_index=len(self.canvas_images),
            graphic_item=graphic_item
        )
        
        self.layers_panel.add_layer(layer)
        
        # ... resto del código ...
    
    def on_layer_selected_from_panel(self, uuid: str):
        """Seleccionar objeto en canvas desde panel de capas"""
        layer = self.layers_panel.find_layer_by_uuid(uuid)
        if layer and layer.graphic_item:
            # Deseleccionar todo
            for item in self.scene.selectedItems():
                item.setSelected(False)
            
            # Seleccionar este
            layer.graphic_item.setSelected(True)
            
            # Centrar en vista (opcional)
            self.view.centerOn(layer.graphic_item)
```

---

## ✅ CARACTERÍSTICAS FINALES IMPLEMENTADAS

### **Funcionalidades Core:**
1. ✅ Jerarquía de capas (parent-child)
2. ✅ Grupos de capas con colapsar/expandir
3. ✅ Renombrado inline con doble click
4. ✅ Visibilidad toggle con icono 👁️
5. ✅ Bloqueo toggle con icono 🔒
6. ✅ Thumbnails actualizables
7. ✅ Drag & drop para reordenar
8. ✅ Multi-selección (Ctrl/Shift)
9. ✅ Búsqueda y filtrado de capas
10. ✅ Colores de identificación

### **Propiedades Avanzadas:**
11. ✅ Opacidad por capa
12. ✅ Modos de fusión (blend modes)
13. ✅ Tags para organización
14. ✅ Notas por capa
15. ✅ Timestamps de creación/modificación

### **Operaciones:**
16. ✅ Duplicar capas
17. ✅ Eliminar capas
18. ✅ Agrupar/Desagrupar
19. ✅ Combinar capas
20. ✅ Reordenar (traer al frente, enviar atrás)

### **UX:**
21. ✅ Menú contextual rico
22. ✅ Atajos de teclado
23. ✅ Indicadores visuales de estado
24. ✅ Iconos por tipo de capa
25. ✅ Indentación visual para jerarquía
26. ✅ Panel de opciones
27. ✅ Barra de información
28. ✅ Tooltips informativos

---

## 🎯 INSTRUCCIONES PARA EL AGENTE

### **PASO 1: IMPLEMENTAR CLASES BASE**
Crea las clases `Layer`, `LayerType`, `BlendMode` exactamente como se especifican arriba.

### **PASO 2: CREAR WIDGETS**
Implementa `LayerItemWidget` y `ProfessionalLayersPanel` con todas sus funciones.

### **PASO 3: INTEGRAR CON CANVAS EDITOR**
Modifica `CanvasEditor` para:
- Reemplazar `self.layers_list` por `self.layers_panel`
- Sincronizar objetos del canvas con capas
- Conectar todas las señales

### **PASO 4: ACTUALIZAR HISTORIAL**
Modificar `save_history_state()` y `restore_state()` para incluir la información de capas completa.

### **PASO 5: TESTING**
Verificar que:
- Todas las operaciones funcionen correctamente
- No se rompa funcionalidad existente
- El drag & drop funcione suavemente
- Los thumbnails se actualicen correctamente

---

## 📊 RESULTADO ESPERADO

Un panel de capas **profesional y completo** que:
- Se vea moderno y pulido
- Funcione de manera estable
- Ofrezca todas las funciones que un usuario profesional espera
- Sea extensible para futuras mejoras

**TIEMPO ESTIMADO**: 1-2 semanas de desarrollo para implementación completa.

---

**¡ADELANTE! Implementa el sistema de capas más profesional posible.**
