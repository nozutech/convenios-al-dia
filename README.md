# Convenios al día

Te avisa por Telegram cuando el BOE o el BOCM publican algo de los convenios colectivos que sigues: un convenio nuevo, tablas salariales, una revisión, una prórroga o un acta de la comisión paritaria. Si ese día no sale nada de tu lista, no te escribe.

Está pensado para quien lleva convenios en una gestoría o en un departamento de personal y hoy los repasa a mano. Es un flujo de n8n. Solo usa datos abiertos, así que no necesita ninguna clave de pago ni base de datos.

## Qué te llega

Este es un aviso real, del BOE del 19/02/2026, con «industria química» en la lista:

> **19/02/2026: novedad en uno de tus convenios**
>
> **Industria química** (BOE): Revisión salarial\
> Acuerdo de revisión salarial del XXI Convenio colectivo general de la industria química.\
> [Ver en el BOE](https://www.boe.es/diario_boe/txt.php?id=BOE-A-2026-3870) | [PDF](https://www.boe.es/boe/dias/2026/02/19/pdfs/BOE-A-2026-3870.pdf)

El tipo de publicación y los años salen del título. Reconoce:
- convenios nuevos;
- tablas salariales y revisiones salariales;
- prórrogas y modificaciones;
- actas de la comisión paritaria;
- sentencias y laudos;
- correcciones de errores.

Lo que no reconoce sale como «Novedad».

## Qué cubre

- **BOE:** los convenios estatales del epígrafe «Convenios colectivos de trabajo».
- **BOCM:** los convenios de la Comunidad de Madrid. Se puede apagar.
- **Fuera:** los boletines de otras comunidades y los provinciales.

Se ejecuta de lunes a sábado a las 8:30, hora de Madrid. El BOE del día sale hacia las 7:30.

## Cómo montarlo

Necesitas n8n 2.x y una cuenta de Telegram. Está probado en n8n 2.45 y 2.46.

1. **Crea un bot.**
   - En Telegram, escribe a @BotFather y envía `/newbot`.
   - Sigue los pasos. Al final te da un token: guárdalo.
2. **Saca tu ID de chat.**
   - Escribe cualquier cosa a tu bot.
   - Abre en el navegador `https://api.telegram.org/bot<TOKEN>/getUpdates`, cambiando `<TOKEN>` por el tuyo.
   - Busca `"chat":{"id":`. El número que va detrás es tu ID.
3. **Importa el flujo.**
   - En n8n, crea un flujo vacío.
   - En el menú de los tres puntos, elige «Import» y luego «From file», y abre `workflows/convenios-al-dia.json`.
   - También puedes copiar el contenido del fichero y pegarlo en el lienzo.
4. **Conecta el bot.** Abre el nodo «Enviar a Telegram», crea una credencial de Telegram y pega el token.
5. **Pon tus datos en el nodo «Configuración».** Por ejemplo:

   ```json
   {
     "fecha": "",
     "chats_telegram": ["123456789"],
     "bocm": true,
     "convenios": ["industria química", "enseñanza privada"]
   }
   ```

   - `convenios`: los convenios que sigues.
   - `chats_telegram`: uno o varios ID de chat. Cada uno recibe el aviso.
   - `bocm`: ponlo a `false` si no te interesan los de Madrid.
   - `fecha`: déjala vacía. Solo sirve para probar.
6. **Pruébalo.**
   - Pon `"fecha": "20260219"`, añade «industria química» a la lista y pulsa «Execute workflow». Te debería llegar el aviso de arriba.
   - Después deja `fecha` vacía otra vez.
7. **Actívalo** con «Publish».

## Cómo escribir la lista

Cada nombre se busca en el título de las publicaciones, al principio de palabra y sin tener en cuenta tildes ni mayúsculas: «quimica» encuentra «Química».

Ponlos concretos, porque un nombre corto puede encontrar otros convenios: «artes gráficas» también encuentra el convenio «del ciclo de comercio del papel y artes gráficas».

Para saber cómo aparece un convenio, búscalo en boe.es y copia la parte del título que lo nombra.

## Límites

- Del BOCM se lee su RSS oficial, que solo trae el último boletín.
  - Si un día salen dos boletines, el primero puede escaparse.
  - Con una `fecha` pasada solo se puede probar el BOE.
  - No se usa el sumario en XML porque el robots.txt del BOCM no lo permite.
- Solo lee los títulos. No abre los PDF.

## BOE al día

`workflows/boe-al-dia.json` es otro flujo, más general. Cada mañana:
1. Filtra el BOE por lo que suele mirar una gestoría: impuestos, Seguridad Social, convenios, Estatuto de los Trabajadores…
2. Pide a un modelo de Groq una frase por publicación: qué hace, a quién afecta y si es de relevancia alta, media o baja.
3. Te lo manda por Telegram.

Para montarlo:
- Necesitas una clave de la API de Groq, que tiene plan gratuito. Va en una credencial de Groq en el nodo «Resumir con Groq».
- En el nodo «Configuración» van:
  - el chat, en `chat_telegram`;
  - el modelo, en `modelo_groq`;
  - los filtros: secciones, epígrafes, palabras clave y exclusiones.

Si Groq falla, el resumen sale igual, solo con los títulos, y lo avisa al final.

## Probarlo en tu ordenador

Con Docker:

```bash
docker compose up -d
```

1. Abre `http://127.0.0.1:5678` y crea la cuenta.
2. Importa los flujos de la carpeta `workflows`.

Para pararlo:

```bash
docker compose down
```

## Licencia

MIT. Lo mantiene [NozuTech](https://nozutech.dev).
