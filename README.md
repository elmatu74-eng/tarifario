# Tarifario de la comarca

Landing estática con aranceles de referencia. Los precios están congelados a una fecha base y se actualizan solos cada mes según el índice UVA.

---

## Cómo está armado

```
index.html                              la página entera (HTML, CSS y JS en un archivo)
datos/base.json                         el ancla: fecha y UVA del día en que se fijaron los precios
datos/factor.json                       lo único que reescribe el robot cada mes
scripts/actualizar-factor.mjs           calcula el factor
.github/workflows/actualizar-factor.yml lo corre el 1 de cada mes
```

La lógica es una sola línea: **precio mostrado = precio base × factor**.

Los precios base nunca se tocan. El factor sale de dividir el UVA de hoy por el UVA de la fecha base. Si `factor.json` no se puede leer, la página usa factor 1 y muestra los valores base: nunca queda rota ni en blanco.

---

## Puesta en marcha

Se hace una sola vez y son unos veinte minutos. Todo desde el navegador, no hace falta instalar nada.

### 1. Crear la cuenta y el repositorio

1. Entrá a [github.com](https://github.com) y creá una cuenta si no tenés.
2. Arriba a la derecha, botón **+** → **New repository**.
3. Completá:
   - **Repository name**: `tarifario`
   - **Public** (tiene que ser público: en el plan gratuito, Pages y las tareas programadas solo funcionan en repos públicos)
   - **No** marques "Add a README file"
4. **Create repository**.

### 2. Subir los archivos

En la pantalla que aparece, hacé clic en **uploading an existing file**.

Arrastrá `index.html`, la carpeta `datos` y la carpeta `scripts`. Después, abajo, botón verde **Commit changes**.

> **La carpeta `.github` va aparte.** Empieza con punto, así que Windows y macOS la esconden y el arrastre no la sube. Se crea a mano:
>
> 1. En la página del repositorio: **Add file** → **Create new file**.
> 2. En el campo del nombre escribí exactamente esto, con las barras incluidas:
>    `.github/workflows/actualizar-factor.yml`
>    (al escribir cada barra, GitHub va creando las carpetas solo)
> 3. Abrí el archivo `actualizar-factor.yml` en tu compu con el Bloc de notas, copiá todo el contenido y pegalo en el recuadro grande.
> 4. **Commit changes**.

### 3. Darle permiso de escritura al robot

Sin esto el workflow corre pero no puede guardar nada.

**Settings** → menú izquierdo **Actions** → **General** → bajá hasta **Workflow permissions** → marcá **Read and write permissions** → **Save**.

### 4. Publicar la página

**Settings** → menú izquierdo **Pages** → en **Source** elegí **Deploy from a branch** → **Branch: main**, carpeta **/ (root)** → **Save**.

Esperá dos o tres minutos y recargá esa misma pantalla: arriba va a aparecer la dirección, con la forma `https://TUUSUARIO.github.io/tarifario/`. Entrá y verificá que se vea.

### 5. Fijar la base

Este paso es el que pone el sistema en hora.

1. Pestaña **Actions** (arriba). La primera vez GitHub pide confirmar: **I understand my workflows, go ahead and enable them**.
2. En la lista de la izquierda, **Actualizar factor**.
3. Botón **Run workflow** → **Run workflow**.
4. Esperá un minuto y recargá. Tiene que quedar con tilde verde.
5. Abrí `datos/base.json` en el repositorio: `fecha_base` y `uva_base` ya no deberían decir `null`.

Listo. A partir del 1 del mes que viene se actualiza solo.

---

## Mantenimiento

### Cambiar un precio o agregar un servicio

1. En el repositorio, clic en `index.html`.
2. Ícono del **lápiz** arriba a la derecha.
3. Buscá el bloque `const DATA = [`. Cada renglón es:
   `["Nombre del servicio", "detalle opcional", precioA, precioB, precioC],`
   Sin puntos ni comas dentro de los números, y `null` donde no aplique.
4. **Commit changes**. En un minuto la página se actualiza sola.

> Los precios que escribas ahí son **a la fecha base**, no a hoy. Si el factor ya está en 1,30 y querés que un servicio se muestre a $100.000, tenés que cargar $76.923. Para evitar la cuenta mental, la alternativa es rebasar el sistema: vaciá `fecha_base` y `uva_base` en `base.json` (dejalos en `null`), cargá los precios como querés que se vean hoy y volvé a correr el workflow a mano. Eso fija una base nueva.

### Saber si se rompió

GitHub te manda un mail cuando un workflow falla. No lo ignores: significa que la fuente del UVA no respondió o cambió de formato. Mientras tanto la página sigue mostrando el último factor bueno, así que no es urgente, pero sí es real.

Si el cambio de un mes supera el 15%, el robot **no publica**: abre un *pull request* para que lo revises a mano. Lo vas a ver en la pestaña **Pull requests**. Revisá el valor contra la fuente y, si está bien, **Merge**.

### Dominio propio

Si tenés un dominio, en **Settings** → **Pages** → **Custom domain** lo cargás, y en tu proveedor de dominio apuntás un registro CNAME a `TUUSUARIO.github.io`.

---

## Fuentes y licencia

Los valores base salen del [tarifario nacional de referencia](https://tarifario.org), ajustados −20% por escala regional. Esa obra está bajo [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.es), y esta también, por herencia: si alguien la reusa, tiene que citar y mantener la misma licencia.

El índice UVA se consulta en [ArgentinaDatos](https://argentinadatos.com).
