# LDD UBA — catálogo de ejercicios interactivos

Repositorio fuente de un sitio Hugo que organiza ejercicios y notebooks para cursos de programación, análisis de datos y bases de datos.

> **Estado:** superficie docente en mantenimiento. El sitio y su submódulo de tema fueron ajustados por última vez en marzo de 2025; el despliegue público y todos los enlaces no fueron revalidados en esta actualización.

## Qué ofrece

El catálogo reúne 58 ejercicios organizados en cuatro áreas:

- Python y Pandas;
- introducción a bases de datos;
- modelado relacional y SQL;
- visualización, probabilidad y estadística aplicada.

`content/_index.md` es la entrada editorial y enlaza las páginas individuales bajo `content/notebooks/`.

## Para quién está pensado

- estudiantes que necesitan ejercicios autocontenidos;
- docentes que quieren seleccionar prácticas por tema;
- asistentes o tutores que necesitan una superficie navegable sobre una colección de notebooks.

El sitio es un catálogo y una publicación docente. No sustituye el repositorio autoritativo de una materia ni garantiza que cada ejercicio corresponda a una cursada vigente.

## Ejecutar localmente

El sitio usa Hugo con el tema Techdoc como submódulo.

```bash
git clone --recurse-submodules https://github.com/matuteiglesias/ldd-uba.git
cd ldd-uba
hugo server
```

Si el repositorio ya fue clonado sin submódulos:

```bash
git submodule update --init --recursive
```

Para construir la versión estática:

```bash
hugo --minify
```

## Estructura

```text
config.toml          configuración de Hugo
content/_index.md    portada y catálogo
content/notebooks/   páginas de los ejercicios
themes/techdoc/      tema administrado como submódulo
```

Los enlaces del contenido están configurados bajo la ruta `/ldd-uba/`; revisar `baseURL` y configuración de despliegue antes de publicar en otro dominio o subruta.

## Autoridad y límites

Este repositorio posee la navegación y el contenido web aquí versionado. Los notebooks originales, datasets o consignas pueden pertenecer a otros repositorios o ediciones del curso; cada página debería declarar su procedencia cuando no sea autocontenida.

La compilación del sitio no prueba que un notebook ejecute correctamente en un entorno moderno.

## Criterio de calidad

Un ejercicio publicado debería indicar:

1. objetivo de aprendizaje;
2. conocimientos previos;
3. datos o dependencias;
4. resultado esperado;
5. procedencia y licencia cuando corresponda;
6. edición o fecha de última revisión.

## Próxima revisión útil

- verificar el despliegue y los 58 enlaces;
- identificar notebooks faltantes o duplicados;
- asociar cada página con su repositorio fuente;
- declarar qué ejercicios forman parte de cursos vigentes;
- agregar una comprobación básica de enlaces durante el build.

El nombre `ldd-uba` es apropiado si la colección continúa vinculada a Laboratorio de Datos en UBA; de otro modo convendría un nombre más general antes de abrir el repositorio al público.
