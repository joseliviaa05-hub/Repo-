# Prompt: Mejora y Expansión Completa del Editor de Canvas

## 🎯 Objetivo Principal

Analiza mi programa de editor de canvas en Python y **implementa todas las mejoras, funciones y características que consideres necesarias** para transformarlo en un editor profesional y completo, inspirado en **Canva, Figma, Photopea, Pixlr y editores similares**.

## 📋 Contexto del Programa

- **Usuario GitHub**: joseliviaa05-hub
- **Programa actual**: Editor de canvas en Python con:
  - Sistema de handles para transformación de objetos
  - Drag and drop de imágenes desde panel lateral
  - Selección y manipulación básica de imágenes

## 🚀 Directrices Generales

**LIBERTAD CREATIVA TOTAL**: Tienes autorización completa para:
1. ✅ Implementar TODAS las funciones que consideres útiles
2. ✅ Rediseñar completamente la interfaz si lo ves necesario
3. ✅ Agregar nuevos módulos y sistemas
4. ✅ Optimizar todo el código existente
5. ✅ Proponer y ejecutar cualquier mejora que beneficie la experiencia de usuario
6. ✅ Añadir funcionalidades profesionales avanzadas
7. ✅ Crear herramientas y utilidades complementarias

**NO HAY LÍMITES**: Si crees que una función va a mejorar el editor, **impleméntala**.

## 🎨 CATEGORÍAS DE MEJORAS REQUERIDAS

### 1. INTERFAZ DE USUARIO (UI) - Inspirado en Canva

```python
"""
OBJETIVO: Crear una interfaz moderna, intuitiva y profesional
"""

# A. LAYOUT PROFESIONAL
class ModernEditorUI:
    """
    Implementar estructura completa tipo Canva:
    
    ┌──────────────────────────────────────────────────────────┐
    │  TOP BAR: Logo | Nombre archivo | Botones acción        │
    ├────┬──────────────────────────────────────────────┬──────┤
    │    │  TOOLBAR: Herramientas principales           │      │
    │    ├──────────────────────────────────────────────┤      │
    │ P  │                                              │  P   │
    │ A  │           CANVAS AREA                        │  A   │
    │ N  │      (Área de trabajo principal)             │  N   │
    │ E  │                                              │  E   │
    │ L  │                                              │  L   │
    │    │                                              │      │
    │ I  ├──────────────────────────────────────────────┤  P   │
    │ Z  │  BOTTOM BAR: Zoom | Coordenadas | Info      │  R   │
    │ Q  └──────────────────────────────────────────────┘  O   │
    │    │                                                 P   │
    │    │  LAYERS PANEL                                  S   │
    └────┴─────────────────────────────────────────────────────┘
    
    IMPLEMENTAR:
    - Top bar con: nombre del proyecto, botones deshacer/rehacer, compartir, exportar
    - Toolbar flotante o fija con todas las herramientas
    - Panel lateral izquierdo: Elementos, Templates, Uploads, Text, etc.
    - Panel lateral derecho: Propiedades del objeto seleccionado
    - Panel de capas (layers) dockeable
    - Barra inferior con zoom slider, coordenadas del mouse, dimensiones del canvas
    - Menús contextuales (click derecho)
    - Tooltips informativos en todos los botones
    """
    
    def __init__(self):
        # Sistema de temas
        self.theme = "dark"  # dark, light, auto
        self.color_scheme = {
            "dark": {
                "bg_primary": "#1e1e1e",
                "bg_secondary": "#252525",
                "bg_tertiary": "#2d2d2d",
                "accent": "#00c4cc",
                "accent_hover": "#00d4dc",
                "text_primary": "#ffffff",
                "text_secondary": "#b0b0b0",
                "border": "#3a3a3a",
                "success": "#4caf50",
                "warning": "#ff9800",
                "error": "#f44336",
            },
            "light": {
                "bg_primary": "#ffffff",
                "bg_secondary": "#f5f5f5",
                "bg_tertiary": "#e0e0e0",
                "accent": "#00a8b8",
                "accent_hover": "#00b8c8",
                "text_primary": "#1e1e1e",
                "text_secondary": "#666666",
                "border": "#cccccc",
                "success": "#4caf50",
                "warning": "#ff9800",
                "error": "#f44336",
            }
        }

# B. COMPONENTES UI MODERNOS
"""
Implementar componentes reutilizables:
- Botones con iconos (usar Pillow o iconos Unicode)
- Sliders para opacidad, rotación, etc.
- Color pickers avanzados
- Dropdowns y comboboxes estilizados
- Tabs para organizar paneles
- Accordions para agrupar opciones
- Toggle switches
- Input fields con validación
- Progress bars para operaciones largas
- Modales y diálogos personalizados
- Notifications/Toasts para feedback
- Loading spinners
"""

# C. ANIMACIONES Y TRANSICIONES
"""
Agregar animaciones suaves:
- Fade in/out de paneles
- Smooth scrolling
- Hover effects con scale y color
- Transiciones entre estados
- Animación al crear nuevos objetos
- Efectos de arrastre fluidos
"""
```

### 2. HERRAMIENTAS DE EDICIÓN - Nivel Profesional

```python
"""
IMPLEMENTAR TODAS ESTAS HERRAMIENTAS
"""

class ToolsManager:
    """
    Sistema completo de herramientas profesionales
    """
    
    # HERRAMIENTAS DE SELECCIÓN
    def __init__(self):
        self.tools = {
            # 1. SELECCIÓN Y TRANSFORMACIÓN
            "select": SelectTool(),          # Selección básica (V)
            "direct_select": DirectSelectTool(),  # Selección de puntos (A)
            "magic_wand": MagicWandTool(),   # Selección por color
            "lasso": LassoTool(),            # Selección libre
            
            # 2. FORMAS Y DIBUJO
            "rectangle": RectangleTool(),    # Rectángulo (R)
            "ellipse": EllipseTool(),        # Círculo/Elipse (O)
            "polygon": PolygonTool(),        # Polígonos
            "star": StarTool(),              # Estrellas
            "line": LineTool(),              # Líneas (L)
            "arrow": ArrowTool(),            # Flechas
            "pen": PenTool(),                # Pluma vectorial (P)
            "pencil": PencilTool(),          # Lápiz libre
            "brush": BrushTool(),            # Pincel (B)
            "eraser": EraserTool(),          # Borrador (E)
            
            # 3. TEXTO
            "text": TextTool(),              # Herramienta de texto (T)
            "text_on_path": TextOnPathTool(), # Texto en curva
            
            # 4. IMÁGENES
            "crop": CropTool(),              # Recortar imagen (C)
            "mask": MaskTool(),              # Máscaras
            "frame": FrameTool(),            # Marcos de imagen
            
            # 5. OTRAS
            "eyedropper": EyedropperTool(),  # Cuentagotas (I)
            "zoom": ZoomTool(),              # Zoom (Z)
            "hand": HandTool(),              # Mano para pan (H)
            "ruler": RulerTool(),            # Regla y medidas
            "gradient": GradientTool(),      # Herramienta de degradado
        }

# IMPLEMENTAR CADA HERRAMIENTA CON:
"""
1. Cursor personalizado
2. Feedback visual durante el uso
3. Configuración de propiedades
4. Shortcuts de teclado
5. Preview en tiempo real
6. Undo/Redo support
"""

# EJEMPLO: Herramienta de Formas Avanzada
class ShapeTool:
    """
    Herramienta para crear formas geométricas
    """
    def __init__(self):
        self.shape_type = "rectangle"  # rectangle, ellipse, polygon, star
        self.fill_color = "#3498db"
        self.stroke_color = "#2c3e50"
        self.stroke_width = 2
        self.corner_radius = 0  # Para rectángulos redondeados
        self.sides = 5  # Para polígonos
        self.points = 5  # Para estrellas
        self.inner_radius = 0.5  # Para estrellas
        
    def properties_panel(self):
        """Panel de propiedades de la forma"""
        return {
            "Fill Color": ColorPicker(),
            "Stroke Color": ColorPicker(),
            "Stroke Width": Slider(0, 20),
            "Corner Radius": Slider(0, 100),
            "Opacity": Slider(0, 100),
            "Blend Mode": Dropdown(["Normal", "Multiply", "Screen", "Overlay"]),
        }
```

### 3. SISTEMA DE CAPAS (LAYERS) - Fundamental

```python
"""
Sistema completo de capas tipo Photoshop/Figma
"""

class LayersPanel:
    """
    Panel de capas profesional
    
    Características:
    - Jerarquía de capas (parent-child)
    - Grupos de capas
    - Orden Z (traer al frente, enviar atrás)
    - Visibilidad (show/hide)
    - Bloqueo de capas
    - Opacidad por capa
    - Modos de fusión (blend modes)
    - Thumbnails de preview
    - Búsqueda y filtrado
    - Renombrado inline
    - Drag and drop para reordenar
    - Multi-selección de capas
    - Duplicar, eliminar, combinar
    - Capas de ajuste
    - Smart objects
    """
    
    def __init__(self):
        self.layers = []
        self.selected_layers = []
        self.layer_groups = []
    
    class Layer:
        def __init__(self, name, layer_type):
            self.id = uuid.uuid4()
            self.name = name
            self.type = layer_type  # image, shape, text, group
            self.visible = True
            self.locked = False
            self.opacity = 100
            self.blend_mode = "normal"
            self.parent = None
            self.children = []
            self.thumbnail = None
            self.z_index = 0
            
        def duplicate(self):
            """Duplica la capa"""
            pass
            
        def merge_down(self):
            """Combina con capa inferior"""
            pass
```

### 4. MANIPULACIÓN DE TEXTO - Completo

```python
"""
Sistema de texto profesional tipo Canva
"""

class TextSystem:
    """
    Sistema completo de edición de texto
    
    FUNCIONALIDADES:
    """
    
    # A. PROPIEDADES BÁSICAS
    class TextObject:
        def __init__(self):
            self.text = ""
            self.font_family = "Arial"
            self.font_size = 16
            self.font_weight = "normal"  # normal, bold, light, etc.
            self.font_style = "normal"   # normal, italic
            self.color = "#000000"
            self.alignment = "left"      # left, center, right, justify
            self.line_height = 1.2
            self.letter_spacing = 0
            self.text_decoration = "none"  # none, underline, line-through
            self.text_transform = "none"   # none, uppercase, lowercase, capitalize
    
    # B. CARACTERÍSTICAS AVANZADAS
    """
    - Editor de texto enriquecido (rich text)
    - Múltiples estilos en un texto
    - Lista de fuentes con preview
    - Búsqueda de fuentes
    - Fuentes de Google Fonts integradas
    - Efectos de texto: sombra, contorno, degradado
    - Texto curvo (text on path)
    - Texto en forma
    - Cajas de texto auto-ajustables
    - Columnas de texto
    - Texto vertical
    - Viñetas y numeración
    - Hipervínculos
    """
    
    # C. EFECTOS DE TEXTO
    class TextEffects:
        """
        Efectos aplicables al texto
        """
        def __init__(self):
            self.shadow = {
                "enabled": False,
                "offset_x": 2,
                "offset_y": 2,
                "blur": 4,
                "color": "#00000080"
            }
            self.stroke = {
                "enabled": False,
                "width": 2,
                "color": "#000000"
            }
            self.gradient = {
                "enabled": False,
                "type": "linear",  # linear, radial
                "colors": ["#ff0000", "#0000ff"],
                "angle": 45
            }
            self.background = {
                "enabled": False,
                "color": "#ffffff",
                "padding": 10,
                "radius": 5
            }
```

### 5. FILTROS Y EFECTOS DE IMAGEN

```python
"""
Sistema completo de filtros y efectos
"""

class ImageFilters:
    """
    Filtros y ajustes profesionales
    
    CATEGORÍAS:
    """
    
    # A. AJUSTES BÁSICOS
    def brightness(self, value):  # -100 a 100
        """Ajustar brillo"""
        pass
    
    def contrast(self, value):  # -100 a 100
        """Ajustar contraste"""
        pass
    
    def saturation(self, value):  # -100 a 100
        """Ajustar saturación"""
        pass
    
    def hue(self, value):  # 0 a 360
        """Rotar tono"""
        pass
    
    def exposure(self, value):  # -2 a 2
        """Ajustar exposición"""
        pass
    
    def temperature(self, value):  # Cálido/Frío
        """Temperatura de color"""
        pass
    
    def tint(self, value):  # Verde/Magenta
        """Tinte de color"""
        pass
    
    # B. EFECTOS DE COLOR
    def grayscale(self):
        """Convertir a escala de grises"""
        pass
    
    def sepia(self):
        """Efecto sepia"""
        pass
    
    def invert(self):
        """Invertir colores"""
        pass
    
    def posterize(self, levels):
        """Posterizar"""
        pass
    
    def threshold(self, value):
        """Umbral blanco/negro"""
        pass
    
    # C. FILTROS ARTÍSTICOS
    def blur(self, radius):  # Gaussian blur
        """Desenfocar"""
        pass
    
    def sharpen(self, amount):
        """Enfocar"""
        pass
    
    def pixelate(self, size):
        """Pixelar"""
        pass
    
    def oil_painting(self):
        """Efecto pintura al óleo"""
        pass
    
    def sketch(self):
        """Efecto boceto"""
        pass
    
    def cartoon(self):
        """Efecto caricatura"""
        pass
    
    # D. CORRECCIONES
    def auto_levels(self):
        """Niveles automáticos"""
        pass
    
    def auto_contrast(self):
        """Contraste automático"""
        pass
    
    def auto_color(self):
        """Color automático"""
        pass
    
    def remove_background(self):
        """Eliminar fondo (IA)"""
        pass
    
    # E. EFECTOS ESPECIALES
    def glow(self, intensity, color):
        """Efecto resplandor"""
        pass
    
    def vignette(self, amount):
        """Viñeta"""
        pass
    
    def lens_flare(self, position):
        """Destello de lente"""
        pass
    
    def motion_blur(self, angle, distance):
        """Desenfoque de movimiento"""
        pass
```

### 6. SISTEMA DE HISTORIAL (UNDO/REDO)

```python
"""
Sistema robusto de historial
"""

class HistoryManager:
    """
    Sistema de undo/redo profesional
    
    CARACTERÍSTICAS:
    - Historial ilimitado (configurable)
    - Panel de historial visual
    - Snapshot de estados
    - Acciones combinables
    - Memory management eficiente
    - Shortcuts: Ctrl+Z, Ctrl+Y, Ctrl+Shift+Z
    """
    
    def __init__(self, max_history=100):
        self.undo_stack = []
        self.redo_stack = []
        self.max_history = max_history
        self.current_state = None
        
    class Action:
        """Acción reversible"""
        def __init__(self, name, do_func, undo_func, data):
            self.name = name
            self.do_func = do_func
            self.undo_func = undo_func
            self.data = data
            self.timestamp = datetime.now()
    
    def add_action(self, action):
        """Agrega acción al historial"""
        pass
    
    def undo(self):
        """Deshacer última acción"""
        pass
    
    def redo(self):
        """Rehacer acción"""
        pass
    
    def clear_history(self):
        """Limpiar historial"""
        pass
    
    def get_history_panel(self):
        """Panel visual de historial"""
        return HistoryPanel(self)

class HistoryPanel:
    """
    Panel que muestra lista de acciones
    - Click en cualquier punto para volver a ese estado
    - Thumbnails de preview
    - Agrupación de acciones similares
    """
    pass
```

### 7. ALINEACIÓN Y DISTRIBUCIÓN

```python
"""
Sistema de alineación profesional
"""

class AlignmentTools:
    """
    Herramientas de alineación y distribución
    
    ALINEACIÓN:
    - Alinear izquierda, centro, derecha
    - Alinear arriba, medio, abajo
    - Alinear al canvas
    - Alinear a selección
    - Alinear a objeto clave
    
    DISTRIBUCIÓN:
    - Distribuir horizontalmente
    - Distribuir verticalmente
    - Espaciado uniforme
    - Distribuir en grid
    
    ORGANIZACIÓN:
    - Agrupar objetos (Ctrl+G)
    - Desagrupar (Ctrl+Shift+G)
    - Traer al frente (Ctrl+Shift+])
    - Enviar atrás (Ctrl+Shift+[)
    - Traer adelante (Ctrl+])
    - Enviar atrás (Ctrl+[)
    """
    
    def align_left(self, objects):
        """Alinea objetos al borde izquierdo"""
        pass
    
    def align_center_horizontal(self, objects):
        """Centra horizontalmente"""
        pass
    
    def distribute_horizontal(self, objects):
        """Distribuye uniformemente en horizontal"""
        pass
    
    def smart_guides(self):
        """
        Guías inteligentes que aparecen al mover objetos:
        - Distancia entre objetos
        - Alineación con otros objetos
        - Centro del canvas
        - Bordes del canvas
        - Snap automático
        """
        pass
```

### 8. GUÍAS Y REGLAS

```python
"""
Sistema de guías y reglas profesional
"""

class GuidesAndRulers:
    """
    Sistema completo de guías
    
    CARACTERÍSTICAS:
    - Reglas horizontales y verticales
    - Guías arrastrables desde reglas
    - Guías personalizadas (click para crear)
    - Snap to guides
    - Grid (cuadrícula)
    - Snap to grid
    - Configuración de grid (tamaño, subdivisiones)
    - Mostrar/ocultar guías (Ctrl+;)
    - Bloquear guías
    - Guías de márgenes
    - Guías de columnas
    - Guías isométricas
    - Unidades: px, cm, mm, in, pt
    """
    
    def __init__(self):
        self.rulers_visible = True
        self.guides = []
        self.grid_enabled = False
        self.grid_size = 20
        self.snap_to_grid = False
        self.snap_to_guides = True
        self.snap_threshold = 5  # pixels
    
    def add_guide(self, orientation, position):
        """Agrega guía horizontal o vertical"""
        pass
    
    def remove_guide(self, guide_id):
        """Elimina guía"""
        pass
    
    def show_grid(self):
        """Muestra cuadrícula"""
        pass
```

### 9. ZOOM Y NAVEGACIÓN

```python
"""
Sistema avanzado de zoom y navegación
"""

class ZoomAndNavigation:
    """
    Controles de zoom profesionales
    
    FUNCIONALIDADES:
    - Zoom in/out (Ctrl + / Ctrl -)
    - Zoom to fit (Ctrl+0)
    - Zoom 100% (Ctrl+1)
    - Zoom 200% (Ctrl+2)
    - Zoom a selección
    - Zoom con scroll del mouse (Ctrl+Scroll)
    - Pan con mano (Space+Drag o Middle-click drag)
    - Mini-map / Navigator panel
    - Zoom slider en barra inferior
    - Porcentaje de zoom editable
    - Smooth zoom con animación
    - Zoom focal (zoom hacia el cursor)
    """
    
    def __init__(self):
        self.zoom_level = 1.0  # 100%
        self.min_zoom = 0.01   # 1%
        self.max_zoom = 64.0   # 6400%
        self.zoom_steps = [0.01, 0.02, 0.03, 0.05, 0.067, 0.08, 0.1, 0.12, 0.125, 0.15, 0.16, 0.2, 0.25, 0.33, 0.5, 0.67, 1, 1.5, 2, 3, 4, 5, 8, 16, 32, 64]
        self.pan_x = 0
        self.pan_y = 0
    
    def zoom_in(self, focal_point=None):
        """Aumenta zoom"""
        pass
    
    def zoom_to_fit(self):
        """Ajusta canvas a ventana"""
        pass
    
    def center_canvas(self):
        """Centra canvas en viewport"""
        pass
```

### 10. EXPORTACIÓN Y GUARDADO

```python
"""
Sistema completo de exportación
"""

class ExportSystem:
    """
    Exportación profesional
    
    FORMATOS DE EXPORTACIÓN:
    - PNG (con transparencia)
    - JPG/JPEG (calidad ajustable)
    - SVG (vectorial)
    - PDF (para impresión)
    - GIF (animado opcional)
    - WEBP
    - BMP
    - TIFF
    - ICO (iconos)
    
    OPCIONES:
    - Resolución personalizada
    - Calidad de compresión
    - Solo selección
    - Múltiples páginas
    - Exportar capas separadas
    - Exportar en lote
    - Presets: Web, Redes Sociales, Impresión
    - Redimensionar al exportar
    - Agregar marca de agua
    
    GUARDADO DE PROYECTO:
    - Formato nativo (.canvas o JSON)
    - Auto-guardado
    - Versiones del proyecto
    - Guardar como template
    """
    
    def export_png(self, path, transparent=True, quality=100, scale=1.0):
        """Exporta como PNG"""
        pass
    
    def export_for_social(self, platform):
        """
        Exporta con dimensiones óptimas:
        - Instagram Post: 1080x1080
        - Instagram Story: 1080x1920
        - Facebook Post: 1200x630
        - Twitter Header: 1500x500
        - YouTube Thumbnail: 1280x720
        """
        pass
    
    def save_project(self, path):
        """Guarda proyecto completo"""
        pass
    
    def load_project(self, path):
        """Carga proyecto"""
        pass
```

### 11. PLANTILLAS Y ASSETS

```python
"""
Sistema de plantillas y recursos
"""

class TemplatesAndAssets:
    """
    Biblioteca de recursos profesional
    
    PLANTILLAS:
    - Plantillas prediseñadas (por categoría)
    - Dimensiones predefinidas (Social media, Documentos, etc.)
    - Crear desde plantilla
    - Guardar como plantilla
    - Importar plantillas
    
    ELEMENTOS:
    - Formas básicas
    - Iconos (biblioteca integrada)
    - Stickers
    - Ilustraciones
    - Fondos
    - Texturas
    - Patrones
    - Marcos (frames)
    - Badges
    
    FOTOS DE STOCK:
    - Integración con APIs de fotos gratuitas:
      - Unsplash
      - Pexels
      - Pixabay
    - Búsqueda de imágenes
    - Preview y descarga
    
    GRÁFICOS:
    - Gráficos de barras
    - Gráficos circulares
    - Líneas de tiempo
    - Infografías
    """
    
    def __init__(self):
        self.categories = [
            "Social Media",
            "Presentations",
            "Posters",
            "Flyers",
            "Business Cards",
            "Logos",
            "Invitations",
            "Resumes",
            "Certificates",
            "Thumbnails",
        ]
    
    def load_template(self, template_id):
        """Carga plantilla"""
        pass
    
    def search_stock_photos(self, query):
        """Busca fotos de stock"""
        pass
```

### 12. COLABORACIÓN Y COMPARTIR

```python
"""
Funcionalidades de colaboración
"""

class CollaborationFeatures:
    """
    Características colaborativas
    
    FUNCIONALIDADES:
    - Compartir proyecto (link público)
    - Exportar como link para ver
    - Comentarios en el diseño
    - Modo presentación
    - Compartir en redes sociales directamente
    - Enviar por email
    - Generar QR code del diseño
    - Embed code para web
    """
    
    def generate_share_link(self, permissions="view"):
        """Genera link de compartir"""
        pass
    
    def export_as_html(self):
        """Exporta como página web embebida"""
        pass
```

### 13. ATAJOS DE TECLADO

```python
"""
Sistema completo de shortcuts
"""

class KeyboardShortcuts:
    """
    Todos los shortcuts profesionales
    
    SHORTCUTS PRINCIPALES:
    """
    shortcuts = {
        # Archivo
        "Ctrl+N": "Nuevo",
        "Ctrl+O": "Abrir",
        "Ctrl+S": "Guardar",
        "Ctrl+Shift+S": "Guardar como",
        "Ctrl+E": "Exportar",
        
        # Edición
        "Ctrl+Z": "Deshacer",
        "Ctrl+Y": "Rehacer",
        "Ctrl+X": "Cortar",
        "Ctrl+C": "Copiar",
        "Ctrl+V": "Pegar",
        "Ctrl+D": "Duplicar",
        "Delete": "Eliminar",
        
        # Selección
        "Ctrl+A": "Seleccionar todo",
        "Ctrl+Shift+A": "Deseleccionar",
        "Ctrl+Click": "Agregar a selección",
        
        # Capas
        "Ctrl+G": "Agrupar",
        "Ctrl+Shift+G": "Desagrupar",
        "Ctrl+]": "Traer adelante",
        "Ctrl+[": "Enviar atrás",
        "Ctrl+Shift+]": "Traer al frente",
        "Ctrl+Shift+[": "Enviar al fondo",
        
        # Vista
        "Ctrl++": "Zoom in",
        "Ctrl+-": "Zoom out",
        "Ctrl+0": "Zoom to fit",
        "Ctrl+1": "Zoom 100%",
        "Ctrl+;": "Mostrar/ocultar guías",
        "Ctrl+'": "Mostrar/ocultar grid",
        
        # Herramientas
        "V": "Selección",
        "T": "Texto",
        "R": "Rectángulo",
        "O": "Elipse",
        "L": "Línea",
        "P": "Pluma",
        "B": "Pincel",
        "E": "Borrador",
        "I": "Cuentagotas",
        "H": "Mano",
        "Z": "Zoom",
        "C": "Recortar",
        
        # Transformación
        "Shift": "Mantener proporciones",
        "Alt": "Desde centro",
        "Ctrl": "Duplicar mientras arrastra",
        
        # Otros
        "F11": "Pantalla completa",
        "Tab": "Ocultar/mostrar paneles",
        "Space": "Mano temporal",
    }
```

### 14. OPTIMIZACIONES TÉCNICAS

```python
"""
Optimizaciones de rendimiento
"""

class PerformanceOptimizations:
    """
    Mejoras de performance
    
    IMPLEMENTAR:
    1. Canvas Rendering
       - Usar offscreen canvas para objetos complejos
       - Cache de renders
       - Dirty rectangles (solo redibujar lo que cambió)
       - RequestAnimationFrame para animaciones
       - Throttle/Debounce de eventos
    
    2. Memory Management
       - Lazy loading de imágenes
       - Comprimir datos en memoria
       - Liberar recursos no usados
       - Virtual scrolling en paneles largos
    
    3. File Handling
       - Carga asíncrona de archivos
       - Progress bars para operaciones largas
       - Chunked loading para archivos grandes
       - Background processing
    
    4. Image Processing
       - Usar PIL/Pillow eficientemente
       - Thumbnails cacheados
       - Resize inteligente
       - Lazy apply de filtros
    
    5. UI Responsiveness
       - Threading para operaciones pesadas
       - Non-blocking UI
       - Feedback visual inmediato
       - Cancelación de operaciones largas
    """
    
    @staticmethod
    def optimize_image(image, max_size=2048):
        """Optimiza imagen para mejor performance"""
        pass
    
    @staticmethod
    def cache_render(object_id, rendered_data):
        """Cachea render de objeto"""
        pass
```

### 15. CARACTERÍSTICAS ADICIONALES

```python
"""
Otras características útiles
"""

# A. BÚSQUEDA GLOBAL
class GlobalSearch:
    """Buscar en todo el editor (Ctrl+K o Ctrl+F)"""
    pass

# B. PANEL DE PROPIEDADES CONTEXTUAL
class PropertiesPanel:
    """
    Panel que muestra propiedades del objeto seleccionado:
    - Posición (X, Y)
    - Tamaño (W, H)
    - Rotación
    - Opacidad
    - Radio de esquinas
    - Color de relleno
    - Borde
    - Sombra
    - Efectos
    - Blend mode
    """
    pass

# C. PÁGINAS MÚLTIPLES
class MultiPageSupport:
    """
    Soporte para múltiples páginas/artboards:
    - Crear nueva página
    - Duplicar página
    - Eliminar página
    - Reordenar páginas
    - Navegación entre páginas
    - Export all pages
    """
    pass

# D. ANIMACIONES Y TRANSICIONES
class AnimationSystem:
    """
    Sistema básico de animación:
    - Fade in/out
    - Slide
    - Scale
    - Rotate
    - Timeline simple
    - Export as GIF/video
    """
    pass

# E. MODO PRESENTACIÓN
class PresentationMode:
    """
    Modo para presentar diseños:
    - Fullscreen
    - Navegación con flechas
    - Zoom en áreas
    - Anotaciones temporales
    """
    pass

# F. MODO OSCURO/CLARO
class ThemeManager:
    """
    Cambio entre modo oscuro y claro
    - Auto detectar sistema
    - Toggle manual
    - Preservar preferencia
    """
    pass

# G. CONFIGURACIONES
class Settings:
    """
    Panel de configuración:
    - Preferencias generales
    - Atajos personalizables
    - Auto-guardado
    - Idioma
    - Unidades por defecto
    - Grid settings
    - Canvas background
    - Performance settings
    """
    pass

# H. TUTORIAL INTERACTIVO
class OnboardingTutorial:
    """
    Tutorial para nuevos usuarios:
    - Tooltips contextuales
    - Walkthrough de herramientas
    - Ejemplos interactivos
    - Skip option
    """
    pass

# I. PLUGINS/EXTENSIONES
class PluginSystem:
    """
    Sistema de plugins (futuro):
    - API para plugins
    - Marketplace de plugins
    - Custom filters
    - Custom tools
    """
    pass
```

## 📊 PRIORIZACIÓN DE IMPLEMENTACIÓN

```
PRIORIDAD ALTA (Implementar primero):
1. ✅ UI moderna y profesional
2. ✅ Sistema de capas completo
3. ✅ Undo/Redo robusto
4. ✅ Herramientas de forma (rectángulo, círculo, etc.)
5. ✅ Sistema de texto completo
6. ✅ Exportación profesional
7. ✅ Zoom y navegación
8. ✅ Alineación y guías

PRIORIDAD MEDIA:
9. ✅ Filtros y efectos de imagen
10. ✅ Panel de propiedades
11. ✅ Plantillas básicas
12. ✅ Atajos de teclado
13. ✅ Modo oscuro/claro
14. ✅ Multi-selección avanzada

PRIORIDAD BAJA (Nice to have):
15. ✅ Animaciones
16. ✅ Colaboración
17. ✅ Plugins
18. ✅ Tutorial interactivo
```

## 🎨 INSPIRACIÓN DE DISEÑO

```
Tomar inspiración de:
- Canva: UI intuitiva, paneles organizados, templates
- Figma: Herramientas profesionales, propiedades detalladas
- Photopea: Filtros y efectos avanzados
- Photoshop: Capas, máscaras, ajustes
- Pixlr: Simplicidad y accesibilidad
```

## 📦 ESTRUCTURA DE ARCHIVOS SUGERIDA

```
canvas_editor/
├── main.py                     # Punto de entrada
├── config.py                   # Configuración global
├── requirements.txt            # Dependencias
│
├── ui/
│   ├── __init__.py
│   ├── main_window.py          # Ventana principal
│   ├── toolbar.py              # Barra de herramientas
│   ├── panels/
│   │   ├── layers_panel.py
│   │   ├── properties_panel.py
│   │   ├── library_panel.py
│   │   ├── templates_panel.py
│   │   └── history_panel.py
│   ├── components/
│   │   ├── color_picker.py
│   │   ├── slider.py
│   │   ├── button.py
│   │   └── modal.py
│   └── themes.py               # Sistema de temas
│
├── canvas/
│   ├── __init__.py
│   ├── canvas_widget.py        # Canvas principal
│   ├── renderer.py             # Motor de renderizado
│   └── viewport.py             # Control de zoom/pan
│
├── objects/
│   ├── __init__.py
│   ├── base_object.py          # Clase base
│   ├── image_object.py
│   ├── shape_object.py
│   ├── text_object.py
│   ├── group_object.py
│   └── layer.py
│
├── tools/
│   ├── __init__.py
│   ├── tool_manager.py
│   ├── select_tool.py
│   ├── shape_tools.py
│   ├── text_tool.py
│   ├── crop_tool.py
│   └── transform_tool.py
│
├── effects/
│   ├── __init__.py
│   ├── filters.py              # Filtros de imagen
│   ├── adjustments.py          # Ajustes de color
│   └── effects.py              # Efectos especiales
│
├── utils/
│   ├── __init__.py
│   ├── math_utils.py
│   ├── color_utils.py
│   ├── file_utils.py
│   ├── image_utils.py
│   └── shortcuts.py
│
├── managers/
│   ├── __init__.py
│   ├── history_manager.py      # Undo/Redo
│   ├── selection_manager.py
│   ├── layer_manager.py
│   └── export_manager.py
│
├── assets/
│   ├── icons/                  # Iconos de herramientas
│   ├── cursors/                # Cursores personalizados
│   ├── templates/              # Plantillas predefinidas
│   └── fonts/                  # Fuentes incluidas
│
└── tests/
    ├── test_canvas.py
    ├── test_tools.py
    └── test_export.py
```

## 🚀 EJECUCIÓN

**IMPORTANTE**: 
- Implementa TODO lo que consideres útil
- No te limites a lo mencionado
- Agrega cualquier función que mejore la experiencia
- Piensa como un diseñador profesional usando el editor
- Prioriza UX y funcionalidad
- Código limpio, modular y bien documentado en español

## ✨ RESULTADO ESPERADO

Un editor de canvas **profesional, completo y usable** que rivalice con editores comerciales, con:
- ✅ Interfaz moderna e intuitiva
- ✅ Todas las herramientas esenciales
- ✅ Performance optimizada
- ✅ Experiencia de usuario fluida
- ✅ Funcionalidades avanzadas
- ✅ Código mantenible y extensible

**¡ADELANTE! Implementa todo lo necesario para crear el mejor editor de canvas en Python.**
