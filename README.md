🚀 Astro 7.3 con JavaScript — Practica lo aprendido (Temas 10, 11 y 12)

En este repositorio se documenta el desarrollo práctico de los ejercicios correspondientes a la Semana 1 · Sesión 2: diferencias entre Astro y JSX (React), las reglas del nuevo compilador en Rust de Astro 7 y la composición de componentes (Nav, Banner y lista de beneficios).

📌 Tema 10: Astro no es JSX

1. Encuentra los errores

Código original con errores (estilo React/JSX):

<label htmlFor="email" className="campo">Correo</label>

¿Por qué falla en Astro?
Astro se apega al estándar de HTML puro y no utiliza atributos en camelCase propios de React como htmlFor o className.

Código corregido en Astro:

<label for="email" class="campo">Correo</label>

2. Dos tipos de comentarios (<!-- --> vs {/\* \*/})

Se agregaron ambos tipos de comentarios dentro de la página y se compiló el proyecto ejecutando pnpm astro build:

<!-- Este comentario es público y llega al navegador -->

{/_ Este comentario es privado y se elimina en el build _/}

Resultado al inspeccionar dist/index.html:

El comentario HTML clásico <!-- ... --> sí aparece en el archivo final generado dentro de dist/ y es visible en el navegador al inspeccionar el código fuente (Ctrl + U o F12).

El comentario estilo JSX {/_ ... _/} no llega al navegador: Astro lo elimina durante la compilación, por lo que sirve para notas internas de desarrollo.

3. Traduce un componente de React (.jsx) a Astro (.astro)

Componente original en React (TarjetaReact.jsx):

export default function TarjetaReact() {
const usuario = "Carlos";
const mostrarAlerta = () => alert("Hola " + usuario);

return (

<div className="tarjeta-usuario" tabIndex={0}>
{/_ Comentario interno _/}
<label htmlFor="apodo" className="etiqueta">Apodo:</label>
<input id="apodo" type="text" />
<button onClick={mostrarAlerta} class="btn">Saludar</button>
</div>
);
}

Versión traducida a Astro (src/components/Tarjeta.astro):
Se eliminó la función envolvente y el return, se movió la lógica del servidor al Frontmatter (---), se cambiaron los atributos a formato HTML estándar (class, tabindex, for) y se reemplazó onClick por una etiqueta <script> del lado del cliente que lee el valor escrito en el <input>:

---

## const usuarioPorDefecto = "Carlos";

<div class="tarjeta-usuario" tabindex="0">
  {/* Comentario privado: no saldrá en el HTML de dist/ */}
  <!-- Comentario público: sí saldrá en el HTML de dist/ -->

<label for="apodo" class="etiqueta">Apodo:</label>
<input id="apodo" type="text" placeholder={usuarioPorDefecto} />

<button id="btn-saludar" data-defecto={usuarioPorDefecto} class="btn">Saludar</button>

</div>

<script>
  const btn = document.getElementById("btn-saludar");
  const inputApodo = document.getElementById("apodo") as HTMLInputElement;

  btn?.addEventListener("click", () => {
    const nombreEscrito = inputApodo.value.trim();
    const nombreFinal = nombreEscrito !== "" ? nombreEscrito : btn.dataset.defecto;
    alert("Hola " + nombreFinal);
  });
</script>

⚙️ Tema 11: Las reglas del compilador de Astro 7

1. Provoca el error (<section> sin cerrar)

Se dejó intencionalmente una etiqueta <section> abierta sin su respectivo cierre </section> en src/pages/index.astro y se ejecutó pnpm astro build.

Mensaje obtenido en la terminal:

[ERROR] Unexpectedly reached the end of the file without closing <section>
Location: src/pages/index.astro:18:3

Análisis: A diferencia de versiones anteriores, el compilador en Rust de Astro 7 detiene el build de inmediato e indica la ruta exacta del archivo (src/pages/index.astro) y el número de línea donde se abrió la etiqueta que nunca fue cerrada. Tras agregar </section>, el proyecto compiló sin errores.

(Opcional: Captura de pantalla del error)
![Error de cierre en section](./public/ErrorDeCierre.png)

2. Inspecciona un anidamiento inválido (<p><div>Hola</div></p>)

Se escribió <p><div>Hola</div></p> y se revisó el resultado en DevTools → Elementos (F12).

¿Cómo quedó el árbol DOM en el navegador?
Dado que en HTML un párrafo <p> solo admite elementos en línea y no admite bloques como <div>, y como Astro 7 ya no "repara" ni altera el HTML mal anidado, el navegador cerró forzosamente el primer <p> antes de abrir el <div> y dejó un <p></p> vacío al final, dividiéndolo en tres elementos hermanos:

<p></p>
<div>Hola</div>
<p></p>

Corrección aplicada:

<div><div>Hola</div></div>
<!-- O usando un elemento en línea válido: -->
<p><span>Hola</span></p>

3. Espacios estilo JSX en dos <span>

En Astro 7 (compressHTML: "jsx"), los saltos de línea entre etiquetas se eliminan, haciendo que dos <span> en líneas separadas se vean pegados ("JuanPérez"). Se corrigió de dos maneras distintas:

Solución 1: Inyectar un espacio explícito con {" "}

<p>
  <span>Juan</span>{" "}
  <span>Pérez</span>
</p>

Solución 2: Colocar ambos elementos en la misma línea

<p><span>Juan</span> <span>Pérez</span></p>

🧩 Tema 12: Práctica de Componentes (Header, Nav, Banner y Beneficios)

1. Componente Nav.astro importado en Header.astro

Se extrajo la etiqueta <nav> hacia su propio archivo reutilizable y se generó el menú dinámicamente con .map().

src/components/Nav.astro:

---

const enlaces = [
{ texto: "Inicio", url: "/" },
{ texto: "Carta", url: "/carta" },
{ texto: "Contacto", url: "/contacto" },
];

---

<nav>
  {enlaces.map((e) => <a href={e.url}>{e.texto}</a>)}
</nav>

src/components/Header.astro:

---

## import Nav from "./Nav.astro";

<header>
  <a href="/">Café Andino</a>
  <Nav />
</header>

2. Banner condicional (Banner.astro)

Muestra el mensaje promocional únicamente cuando la variable promoActiva es true, utilizando evaluación de cortocircuito con &&:

src/components/Banner.astro:

---

## const promoActiva = true;

{promoActiva && (

  <div class="banner">
    <p>Hoy 2x1 en capuchinos</p>
  </div>
)}

3. Sección de beneficios con .map() y Fragmentos (<>...</>)

Dentro de src/pages/index.astro, se recorrió un arreglo de 4 beneficios utilizando una lista de definición (<dl>) y agrupando los pares <dt> y <dd> mediante un fragmento invisible (<>...</>) para no ensuciar el HTML con etiquetas <div> inválidas:

---

const beneficios = [
{ titulo: "Envío gratis", detalle: "En todos tus pedidos mayores a S/ 50." },
{ titulo: "Café de altura", detalle: "Cultivado a más de 1,800 m s. n. m. en Cajamarca." },
{ titulo: "Tostado fresco", detalle: "Tostamos los granos cada semana." },
{ titulo: "Comercio justo", detalle: "Trato directo con productores locales." },
];

---

<section>
  <h2>Nuestros Beneficios</h2>
  <dl>
    {beneficios.map((b) => (
      <>
        <dt><strong>{b.titulo}</strong></dt>
        <dd>{b.detalle}</dd>
      </>
    ))}
  </dl>
</section>
