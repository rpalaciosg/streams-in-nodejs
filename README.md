# Streams in Node.js

Al igual que los pipes en Unix, los `streams` en node.js nos permiten trasnferir grandes cantidades de datos de forma
rapida y eficiente, ya que divide los datos en fragmentos en lugar de leer todo a la vez en memora.

En este documento aprenderemos a usar los metodos `fs.createReadStream`, `fs.createWriteStream` y `Pipe` para procesar
archivos JSON y CSV grandes (grandes cantidades de datos).

Las streams son otra forma de manejar lecturas y escritura de archivos en Node.js, los streams representan un flujo de datos asincronos.

Estan pensados para funcionar como los `pipes en unix`, permite conectar una fuente de datos en un extremo, con cualquier numero de consumidores al otro extremo y poder pasar datos entre ellos.
Estos ayuda a procesar archivos muy grandes, ya no leen todo en memoria una sola vez sino que procesa en trozos mas pequeños. Todo esto usando el modulo `fs` de node.js.

## Que es un Stream en node.js?

Son una interfaz para trabajar con datos de streaming( datos que llegan fragmento a fragmento a lo largo del tiempo). Es parecido a comofunciona los pipes (|) de Unix.

Basicamente son una coleccion de datos que no estan disponibles todos a la vez, debemos manejar cada fragmento de dato a medida que llega del `stream`.

En node, muchos modulos integrados usan `streams` para el procesamiento asincrono de datos, como los metodos `ClientRequest` y `ServerResponse` del modulo `http`.

Los datos del `stream` son `Buffer` (secuencias de bytes de longitud fija) de forma predeterminada, a menos que se configure para trabajar con objetos.

### Porque usar streams?

-   Eficiencia de memoria, cuando trabajamos con datos demasiado grandes para entrar en la memoria, los strems son muy eficientes con el manejo de memoria.
    Si necesitamos trabajar con un archivo de 20GB, y deseamos transformar cada fila en un conjunto de datos, si leemos todo el archivo en memoria, a parte de tomar mucho tiempo, alcanzara el limite de memoria de Node.js dandonos un error de memoria. Al trabajar los streams con fragmentos pequeños de datos, consumen cantidades minimas de memoria.
-   Manejar datos como llegan: los streams no solo son utiles para trabajar con archivos grandes, tambien nos da la ventaja de que al consumir datos, podemos empezar a procesar tan pronto llegue el primer fragmento, y nos ahorraran tiempo en comparacion a esperar que se lean todos los datos en memoria, ahorrandonos tiempo.
-   Ademas los streams se pueden componer dentro de un `pipeline`, es decir se pueden conectar y combinar con otros streams, ya que la salida de un stream se puede usar como entrada para otro stream, a travez de los pipelines los datos pueden fluir a travez de varios o distintos streams.

### Conociendo a los streams

En el modulo `stream` en node.js hay 4 tipos de streams:

-   `Readable`: es una fuente de entrada/input, y recibe datos de un `Readable` stream.
-   `Writable`: Es un destino, trasnmite datos de stream `Writable`, avees se conoce como un `sinc` o receptor porque es el destino final de la transmision de datos.
-   `Duplex`: es un stream que imprementa una interfaz `Readable` y `Writable`. ya que pueden recibir y producir datos. Un ejemplo es un socket TCP, donde los datos fluyen en ambas direcciones.
-   `Transform`: Es un tipo de stream `Duplex` en el que se modifican los datos que pasen a travez de este stream, la salida sera diferente de la entrada. Se puede enviar datos a un stream `Transform` y asi mismo leer datos de ella despues de que se hayan transformado.
    -   `PassThrough`: Es un tipo especial de stream `Transform`, este no altera los datos que pasan o lo atraviesan, segun la documentacion se usan principalmente para pruebas y ejemplos, tiene poco proposito real mas alla de transmitir datos.

De todos estos, los streams que mas nos encontraremos son `Readable`, `Writable` y `Transform`.

### Stream Events

-   Instance of `EventEmitter`
-   No se recomienda interactuar con streams via eventos.
-   Util para controlar streams detalladamente.

### Readable Stream Events

-   data: chunk of data, trozos de datos
-   readable: data lista para ser leida.
-   end: no mas data disponible
-   error: ha ocurrido un error.

### Writable stream Events

-   drain: lista para recibir mas datos.
-   finish: todos los datos escritos en el sistema subyacente
-   error: un error ocurrio

### Conectar Streams con pipe

-   pipe (canalizar) conectar streams juntos
-   pipe (canalizar) datos de Readable interfaces a interfaces Writable.

```javascript
const { PassThrough } = require("stream");
const passThrough = new PassThrough();
process.stdin.pipe(passThrough).pipe(process.stdout);
```

### Crear File streams con fs

-   fs.createReadStream
-   fs.createWriteStream

```javascript
const fs = require("fs");
const outputStream = fs.createWriteStream("file.txt");

process.stdin.pipe(outputStream);
```

```javascript
const fs = require("fs");
const inputFileStream = fs.createReadStream("file.txt");

inputFileStream.pipe(proccess.stdout);
```

### Conectar Streams con pipeline

-   Agregado en la version v10.0.0 de node.js
-   Mejora con respecto al uso de `pipe`

```javascript
const { PassThrough, pipeline } = require("stream");
const fs = require("fs");

const input = fs.createReadStream("file.txt");
const out = fs.createWriteStream("out.txt");

const passThrough = new PassThrough();

console.log("Iniciando pipeline...");
pipeline(input, passThrough, out, (err) => {
    if (err) {
        console.log("Pipeline fallo con un error:", err);
    } else {
        console.log("Pipeline termino satisfactoriamente");
    }
});
```

```javascript
const { Transform, pipeline } = require("stream");

const upperCaseTransform = new Transform({
    transform(chunk, encoding, callback) {
        callback(null, chunk.toString().toUpperCase());
    },
});

pipeline(process.stdin, upperCaseTransform, process.stdout, (err) => {
    if (err) {
        console.log("Pipeline encontro un error", err);
    }
});
```

Si ejecutamos el codigo node ....., y pasamos un stream por ejemplo 'hola mundo', en `trasnform` nos pasara 'HOLA MUNDO' ya que el passThrough es un transform que devuelve los strings en mayusculas.

## Streams para extraer, transformar, y cargar datos en un CSV

## Stream (transmitir) una respuesta HTTP con node.js

## Notas
