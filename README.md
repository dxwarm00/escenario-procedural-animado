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
    # Crea un material usando el sistema de nodos (PBR)
    mat = bpy.data.materials.new(name=nombre)
    mat.use_nodes = True
    nodes = mat.node_tree.nodes
    
    # Buscamos el nodo principal (Principled BSDF)
    bsdf = nodes.get("Principled BSDF")
    if bsdf:
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

<img width="824" height="492" alt="image" src="https://github.com/user-attachments/assets/f850204b-6c17-4f72-9d3d-ee99b7cedaf5" />

<details>
<summary><b>Ver código completo del script (Python) 🐍</b></summary>

```python
import bpy
import math
import random

def crear_material(nombre, color_rgb):
    # Crea un material básico con un color específico
    mat = bpy.data.materials.new(name=nombre)
    mat.diffuse_color = (*color_rgb, 1.0) # RGBA
    return mat

def generar_escenario():
    # 1. Limpiar la escena previa
    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete()

    # 2. Definir materiales (Basado en modelos de color RGB)
    mat_pared_a = crear_material("ParedOscura", (0.1, 0.1, 0.1))
    mat_pared_b = crear_material("ParedDetalle", (0.8, 0.2, 0.0)) # Un naranja rojizo

    # 3. Parámetros del escenario
    largo_pasillo = 10
    ancho_pasillo = 4

    # 4. Ciclo para construir paredes (Transformación: Traslación)
    for i in range(largo_pasillo):
        # Pared Izquierda
        bpy.ops.mesh.primitive_cube_add(location=(-ancho_pasillo, i * 2, 1))
        pared_izq = bpy.context.active_object

        # Aplicar material de forma alternada (Lógica de programación)
        if i % 2 == 0:
            pared_izq.data.materials.append(mat_pared_a)
        else:
            pared_izq.data.materials.append(mat_pared_b)
            # Escalamiento para darle variedad visual
            pared_izq.scale.z = 1.5

        # Pared Derecha
        bpy.ops.mesh.primitive_cube_add(location=(ancho_pasillo, i * 2, 1))
        pared_der = bpy.context.active_object
        pared_der.data.materials.append(mat_pared_a)

    # 5. Agregar un suelo (Escalamiento y Posicionamiento)
    bpy.ops.mesh.primitive_plane_add(size=1, location=(0, (largo_pasillo - 1), 0))
    suelo = bpy.context.active_object
    suelo.scale.x = ancho_pasillo + 1
    suelo.scale.y = largo_pasillo + 1 

generar_escenario()
```
</details>

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

### 3. Explicación Matemática del Tramo Curvo

Para convertir el movimiento lineal del pasillo en una trayectoria circular, se aplicaron **coordenadas polares**. En lugar de simplemente desplazar los objetos en el eje $Y$, calculamos su posición en el plano $XY$ utilizando funciones trigonométricas basadas en un radio central ($R$) y un ángulo variable ($\theta$).

**Las fórmulas aplicadas son:**

$$x = cx + R \cdot \cos(\theta)$$
$$y = cy + R \cdot \sin(\theta)$$

Donde:
* **$cx, cy$**: Es el centro de la circunferencia donde se origina la curva.
* **$R$**: Es el radio de giro (ajustado según si es la pared interna o externa).
* **$\theta$ (theta)**: Es el ángulo en radianes, calculado proporcionalmente al número de piezas dentro del ciclo `for`.


<p align="center">
  <img src="https://github.com/user-attachments/assets/510cba37-c0d5-4444-a04f-ceebf589f39d" width="300">
</p>


Además, para que las paredes no queden "mirando" siempre hacia la misma dirección, se aplicó una **transformación de rotación** en el eje $Z$:
`rotacion_z = math.pi - angulo`, asegurando que cada bloque esté alineado perpendicularmente al radio de la curva.

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

<img width="898" height="514" alt="image" src="https://github.com/user-attachments/assets/60d57753-120b-45a4-9bc2-f2777f883363" />

---

## 💾 Código Fuente del Proyecto

Para mantener este documento legible, el código completo del escenario procedural (incluyendo la lógica de la curva y la animación de cámara) se encuentra en un archivo separado.

👉 **[Ver script completo aquí: escenario_final.py](./escenario_final.py)**

---
