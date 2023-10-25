![UCU](https://github.com/ucudal/PII_Conceptos_De_POO/raw/master/Assets/logo-ucu.png)

### FIT - Universidad Católica del Uruguay

<br>

# Excepciones

La lógica de un programa orientado a objetos la implementamos creando objetos, enviado mensajes a esos objetos, y utilizando estructuras de control de flujo para determinar qué mensajes enviar, a qué objetos, y cuándo. Al enviar un mensaje, esperamos que cambie el estado del programa de forma determinada por el comportamiento del método que se ejecuta como resultado de la recepción del mensaje<sup>1</sup>; si eso no sucede, entonces estamos ante una situación desconocida, típicamente decimos que el programa es incorrecto<sup>2</sup>.

Más formalmente, definimos los siguientes términos (basado en IEEE 610.12 1990):

- Una acción humana que produce un resultado incorrecto. El término más preciso para este significado es equivocación (_mistake_).

- Una definición incorrecta en un paso, un proceso, o un dato; un defecto en un dispositivo o componente. El término más preciso para este significado es falta (_fault_): la manifestación de una equivocación.

- Un comportamiento o resultado incorrecto. La incapacidad de un sistema o componente de realizar las funciones requeridas con el desempeño especificado. El término más preciso es falla (_failure_): el resultado de una falta.

- La diferencia entre un valor o condición calculada, observada, o medida, y el valor o condición verdadera, especificada, o teórica. El término adecuado es error: la cantidad por la que el resultado es incorrecto.

Una equivocación puede derivar en un error. Cuando aparece un error en nuestro programa y como programadores creemos que la equivocación es de quién interactúa con nuestro programa, por ejemplo, el usuario final u otro programa, podemos tratar de manejar ese error y tratar de subsanar la equivocación; ocurre una falta, pero no una falla.

Piensen, por ejemplo, en el caso que nuestro programa tenga que usar un archivo cuya ubicación ingresa el usuario. El usuario puede cometer una equivocación al ingresar una ubicación donde no existe un archivo y nuestro programa no podrá abrirlo; pero si le damos una nueva oportunidad al usuario, podría ingresar esta vez la ubicación correcta, y nuestro programa sí podría abrirlo. O el usuario desiste de abrir el archivo, y nuestro programa termina como se espera.

Cuando trato de abrir un archivo puede ocurrir una excepción de la clase `FileNotFoundException`<sup>3</sup> si la ubicación ingresada por el usuario no existe. Es posible manejar esa excepción para pedir nuevamente la ubicación del archivo al usuario, o permitirle desistir de abrirlo, y el programa termina decorosamente.

También puede aparecer un error en nuestro programa que inadvertidamente es una equivocación nuestra como programadores, o de otro programador que programó una clase que nosotros usamos, o que usa una clase que nosotros programamos. Esto ocurre cuando lo que hace el programa no está de acuerdo con la especificación de lo que debería hacer el programa. En este caso, a diferencia del anterior, no es una situación que se pueda resolver, ocurre una falla, y nuestro programa no puede continuar: la excepción no es manejable.

Cuando trato de enviar un mensaje a una variable en la que no hay un objeto -la variable tiene el valor null- ocurre una excepción de la clase `NullReferenceException`; si esto ocurre cuando está especificado que debería haber un objeto, no es una situación que podamos cambiar sin generar une nueva versión del programa.

La decisión de si una excepción es un falla o no depende del contexto, la toma el objeto que envía el mensaje. Siguiendo con el ejemplo de un programa que usa un archivo, la clase que usamos para abrirlo va a generar una excepción si el archivo no existe, nuestro programa es el que decide pedir nuevamente su ubicación.

Desde el punto de vista del flujo de ejecución del programa, el manejo de excepciones se realiza con una estructura de control llamada `try…catch…finally`. En el bloque `try` están las sentencias de código en las que puede ocurrir una excepción que me interesa. En el bloque `catch` están las sentencias que se ejecutan cuando ocurre una excepción en el bloque `try`; si no ocurre ninguna, no se ejecuta. En el bloque `finally` están las sentencias que quiero asegurarme que se ejecuten siempre, haya ocurrido una excepción o no. Típicamente están asociadas a liberar recursos tomados en el bloque `try`.

Vean el siguiente código, que abre un archivo para escribir en él. Como la apertura y escritura del archivo pueden generar excepciones, están dentro del bloque `try`. Una de las excepciones que me interesa capturar es `UnauthorizedAccessException`, que ocurre cuando no tengo acceso a un archivo; si eso ocurre, se ejecuta el código en el bloque `catch`, que imprime el mensaje "Acceso no autorizado" en la consola de depuración; si tengo acceso al archivo, no se imprimirá nada. Por último, haya ocurrido una excepción o no, quiero asegurarme de que el archivo que abrí quede cerrado, de lo contrario nadie más podrá acceder a él; por eso, hay un bloque `finally` donde se cierra el archivo.

```c#
FileStream file = null;
FileInfo fileinfo = new FileInfo(@"c:\file.txt");

try
{
    file = fileinfo.OpenWrite();
    file.WriteByte(0xF);
}
catch (UnauthorizedAccessException e)
{
    Debug.WriteLine("Acceso no autorizado");
    throw e;
}
finally
{
    if (file != null)
    {
        file.Close();
    }
}
```

En C# todas las excepciones son sucesoras de la clase base `System.Exception`.

Para crear una excepción -también se dice lanzar una excepción-, se utiliza la palabra clave `throw` seguida de la clase de excepción que quiero lanzar; en el bloque `catch` se puede usar `throw` sin argumentos, para volver a lanzar la excepción que se está capturando en ese bloque. El bloque `catch` puede incluir o no una clase de excepción. Cuando no la incluye, cualquier excepción en el bloque `try` provoca la ejecución del bloque `catch`. Cuando la incluye, como en el ejemplo anterior, sólo una excepción de esa clase en el boque `try` provoca la ejecución del bloque `catch`; vean que en este caso se recibe un objeto de la clase de excepción correspondiente, que incluye información adicional sobre la excepción, como un mensaje. Es posible concatenar varios bloques `catch` para manejar excepciones de diferentes clases. Si ocurre una excepción y no hay un bloque `try…catch…finally`, se busca un bloque `try…catch…finally` en el método que invocó al método en el que ocurre la excepción, y así sucesivamente hasta llegar el programa principal. El programa termina si no se maneja la excepción.

## Desarrollo

Recordemos un programa que ha hemos visto antes, la biblioteca representada en la clase `Library`, que puede contener obras musicales representadas por la clase `Media`, que puede ser tanto canciones como películas, representadas en las clases `Song` y `Media` respectivamente. Supongamos que no me gusta cierto género, por ejemplo, la marcha; la especificación de mi programa dice que no pueden existir canciones de ese género, y si alguien intenta crear una, es una violación de la especificación. ¿Cómo puedo evitar que se creen canciones de ese género? Una posibilidad es levantar una excepción cuando se intenta agregar una canción del género marcha. Primero definimos la clase de la nueva excepción a levantar, que llamaremos `LibraryException`. No es absolutamente necesario definir nuevas clases de excepción si las que ya existen son adecuadas, pero en esta caso queremos crearla para mostrarles cómo se hace. Como dijimos antes, todas las excepciones son instancias de una clase sucesora de la clase base `Exception`. Vean el código a continuación:

```c#
[Serializable]
internal class LibraryException : Exception
{
    public LibraryException()
    {
    }
    
    public LibraryException(string message)
    : base(message)
    {
    }
    
    public LibraryException(string message, Exception innerException)
    : base(message, innerException)
    {
    }
    
    protected LibraryException(SerializationInfo info, StreamingContext context)
    : base(info, context)
    {
    }
}
```

> [Ver en repositorio »](https://github.com/ucudal/PII_Exceptions/blob/master/src/Library/LibraryException.cs)

<br/>

Para ser sucesora de `Exception` nuestra `LibraryException` debe implementar constructores sin parámetros, con el mensaje como parámetro, con un mensaje y una excepción encadenada, y debe ser decorada con el atributo `[Serializable]`.

Como el objetivo es evitar crear instancias de `Song` con género "marcha", el constructor tira una excepción si corresponde. Vean el código a continuación, especialmente la sentencia `throw`. Recuerden que la excepción es un objeto, por es necesario usar `new` para crearla.

```c#
public class Song : Media
{
    private string artist;
    
    public Song(string name, string genre, string artist)
    : base(name, genre)
    {
        if (string.Equals(genre, "marcha", StringComparison.OrdinalIgnoreCase))
        {
            throw new LibraryException($"No me gusta {name} porque no me gusta la marcha");
        }
        this.artist = artist;
    }
}
```

> [Ver en repositorio »](https://github.com/ucudal/PII_Exceptions/blob/master/src/Library/Song.cs)

<br/>

El programa principal agrega varias canciones, protegemos la posibilidad de ocurrencia de una excepción en un bloque `try`; si ocurre una excepción, en el bloque `catch` simplemente se imprime por consola el mensaje de la excepción; en cualquier caso, en el bloque `finally`, se imprime al final el contenido de la biblioteca:

```c#
public class Program
{
    private static Library library = new Library();
    
    public static void Main(string[] args)
    {
        try
        {
            AddMovies();
            AddSongs();
            AddMarchaSongs();
        }
        catch (LibraryException e)
        {
            Console.WriteLine(e.Message);
        }
        finally
        {
            PrintMediaItems("La biblioteca contiene:", library);
        }
    }
}
```

> [Ver en repositorio »](https://github.com/ucudal/PII_Exceptions/blob/master/src/Program/Program.cs)

<br/>


******

<sup>1</sup> _Más precisamente, el resultado del envío de un mensaje está determinado por la especificación de ese método._

<sup>2</sup> _Una pieza de software es correcta cuando satisface su especificación. Por lo tanto, la corrección es un concepto relativo: sin especificación no hay corrección._

<sup>3</sup> _La excepción es una instancia de esta clase, por lo tanto es un objeto._
