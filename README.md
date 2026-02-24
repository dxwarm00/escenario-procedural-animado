# Proyecto Integrador – Escenario Procedural (Unidad 1)

## Parte 1 – Escenario Procedural Base

### Objetivo

Construir un escenario 3D mediante scripting en Blender,
aplicando transformaciones tridimensionales (traslación y escalamiento)
y modelos de color RGB.

---

## 1. Limpieza de la escena

Se eliminan todos los objetos previos para comenzar desde cero.

```python
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()
```

Esto asegura que no existan objetos residuales en memoria.

---

## 2. Creación de materiales (Modelo RGB)

Se define una función para generar materiales reutilizables.

```python
def crear_material(nombre, color_rgb):
    mat = bpy.data.materials.new(name=nombre)
    mat.use_nodes = True
    bsdf = mat.node_tree.nodes["Principled BSDF"]
    bsdf.inputs['Base Color'].default_value = (*color_rgb, 1.0)
    return mat
```

Esto permite aplicar el modelo de color RGB y evitar repetición de código.

---

## 3. Construcción del pasillo

Se utiliza un ciclo `for` para automatizar la creación de las paredes.

```python
for i in range(largo_pasillo):
    bpy.ops.mesh.primitive_cube_add(location=(-ancho_pasillo, i * 2, 1))
```

Aquí se aplica la transformación de traslación en el eje Y.

---

## 4. Suelo e iluminación

Se agrega un plano escalado como suelo y una luz tipo POINT
para iluminar el escenario.

El resultado es un pasillo recto generado automáticamente.

---

# Parte 2 – Mejora del Proyecto: Tramo Curvo y Animación de Cámara

## 1. Generación del tramo curvo

Se añadieron cálculos trigonométricos para crear una sección curva
al final del pasillo.

```python
angulo = math.pi - (j * (math.pi / 2) / largo_pasillo)
x = cx + (radio_curva + ancho_pasillo) * math.cos(angulo)
y = cy + (radio_curva + ancho_pasillo) * math.sin(angulo)
```

Esto permite posicionar los cubos en una trayectoria circular.

---

## 2. Creación del camino (Path)

Se creó una curva 3D que funciona como trayectoria de animación.

```python
curve_data = bpy.data.curves.new('CamPathData', type='CURVE')
curve_data.use_path = True
```

---

## 3. Aplicación de Constraints

Se utilizaron las restricciones:

- Follow Path
- Track To

Para que la cámara siga el recorrido y apunte hacia el objetivo.

---

## 4. Animación con Keyframes

Se definieron los fotogramas inicial y final,
y se animó el desplazamiento mediante `offset_factor`.

```python
camera.keyframe_insert(data_path=f'constraints["{follow_path.name}"].offset_factor', frame=1)
```

Esto permite que la cámara recorra todo el escenario de manera automática.

![AnimationEscenario](https://github.com/user-attachments/assets/834081e9-e320-46e6-93c3-9a6c4dce473e)
