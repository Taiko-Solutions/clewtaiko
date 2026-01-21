# Clew en pocas palabras

Clew es una técnica que permite a los desarrolladores de FileMaker implementar fácilmente validaciones y captura y manejo robusto de errores en sus soluciones. Además, clew captura automáticamente un registro de llamadas a scripts y errores, así como información de estado para cada script en la pila de llamadas. Esto puede representar información muy valiosa para propósitos de depuración y registro.

Está compuesto por un conjunto de funciones personalizadas que no tienen dependencias de nada más que de sí mismas. Clew no requiere ningún framework, NO le importa cómo pasas parámetros o devuelves resultados en los scripts, y no requiere ningún esquema, scripts o layouts. Cuando ocurre un error en un script, instancia una y solo una variable local (no es necesaria la limpieza de variables globales después de ejecutar scripts), la cual puedes nombrar como quieras en la función error.Config.

<br>

# Comenzando con clew

### Para empezar a usar clew simplemente:

- Descarga el archivo FileMaker de clew desde la [última versión](https://github.com/soliantconsulting/clew/releases)
  - Copia y pega la carpeta de funciones personalizadas "clew" en tu solución y comienza a usarlas

### Aquí hay algunas referencias para aprender a usar clew:

- Revisa los scripts Demo y Template en el archivo FileMaker, que muestran cómo usar clew
- Revisa la documentación a continuación en este archivo readme
- Mira la [presentación](https://youtu.be/d4N7d0Kdxqs?si=o2sMZicN_oNEzbwz) que di sobre clew en EngageU 2024

<br>

# Funciones de Clew y anatomía de un script de FileMaker que usa clew

El diagrama a continuación representa todas las funciones personalizadas de clew (excepto las funciones de error constantes), y muestra dónde y cómo se utilizan en un script típico

![Representación visual de un pseudo script de FileMaker escrito usando clew](clew_visual_explanation.png)

<br>

# Ejemplo de traza de error generada por clew

A continuación se muestra una traza muy simple generada por clew. Representa una secuencia de scripts en la que el script "Demo script - Terse" llamó al subscript "throws FMP error - Send Mail", y el subscript devolvió un error, el cual el script llamador no manejó.

    {
      "directionOfTrace": "caller_script_last",
      "errorTrace": [
        {
          "code": 1502,
          "description": "Connection refused by SMTP server",
          "hint": "Failed to send notification for monthly reconciliation report",
          "script": {
            "name": "throws FMP error - Send Mail",
            "parameter": null,
            "stepNumber": 7,
            "stepType": "Send Mail"
          },
          "state": {
            "accountName": "Admin",
            "applicationVersion": "Pro 21.1.1",
            "layoutName": "AnotherTable Layout",
            "recordOpenCount": 1
          }
        },
        {
          "code": "UNEXPECTED",
          "description": "Generic, default error",
          "hint": "Unexpected error when calling this subscript: 'throws FMP error - Send Mail'",
          "script": {
            "name": "Demo script - Terse",
            "parameter": {
              "param_one": 15,
              "param_two": "I am a string"
            }
          },
          "state": {
            "accountName": "Admin",
            "applicationVersion": "Pro 21.1.1",
            "layoutName": "DemoTable Layout",
            "recordOpenCount": 0
          }
        }
      ]
    }

<br>

# Esquema de traza de error

La estructura pseudo-json a continuación describe el esquema json al que se adhiere una traza de error de clew:

Las propiedades en $\color{#3399CC}{\textsf{azul}}$ conforman el esquema json básico

Las propiedades en $\color{#009966}{\textsf{verde}}$ son opcionales o personalizables para cualquier solución en la que se utilice la técnica clew

Las propiedades en $\color{orange}{\textsf{naranja}}$ solo se incluyen para errores nativos de FileMaker, ya que se analizan desde la función de FileMaker Get( LastErrorLocation )

{  
&nbsp;&nbsp; $\color{#3399CC}{\textsf{"directionOfTrace"}}$: // e.g. "caller_script_first"  
&nbsp;&nbsp; $\color{#3399CC}{\textsf{"errorTrace"}}$: [ // an array of objects with the below structure  
&nbsp;&nbsp;&nbsp;&nbsp; {  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\color{#3399CC}{\textsf{"code"}}$: // number or string representing an error condition  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\color{#009966}{\textsf{"description"}}$: // an English description of the "code" json property above  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\color{#3399CC}{\textsf{"hint"}}$: // either a string or a json structure with customized info about the error  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\color{#3399CC}{\textsf{"script"}}$: { // object that contains at least a "name" property  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\color{#3399CC}{\textsf{"name"}}$: // name of script, as returned by FileMaker function Get( ScriptName )  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\color{#009966}{\textsf{"parameter"}}$: {} // string or json, as returned by Get( ScriptParameter )  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\color{orange}{\textsf{"stepErrorDetail"}}$: // whatever FileMaker function Get( LastErrorDetail ) returns  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\color{orange}{\textsf{"stepNumber"}}$: // e.g. 89, as returned by Get( LastErrorLocation )  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\color{orange}{\textsf{"stepType"}}$: // e.g. Send Mail, as returned by Get( LastErrorLocation )  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; },  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\color{#009966}{\textsf{"state"}}$: { // optional object, 100% developer customizable  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\color{#009966}{\textsf{"accountName"}}$: // e.g. "Admin"  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\color{#009966}{\textsf{"fileName"}}$: // e.g. "FM_Error_Stack_Trace"  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\color{#009966}{\textsf{"layoutName"}}$: // e.g. "Table 1"  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\color{#009966}{\textsf{"recordOpenCount"}}$: // e.g. 0  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }  
&nbsp;&nbsp;&nbsp;&nbsp; }  
&nbsp;&nbsp; ]  
}  

Nota: las propiedades se nombran en parte de la manera en que se nombran, para asegurar que las propiedades se muestren en el orden deseado, incluso cuando la estructura json se ordena alfabéticamente por un analizador o editor json

<br>

# Agradecimientos

Marcelo Piñeyro, el desarrollador principal de la técnica clew, desea reconocer a las siguientes personas por sus contribuciones a clew:

- Andrew "Kaz" McLamore, por compartir con nosotros su implementación de captura de errores en FileMaker como objetos json, y por usar la técnica de bucle de un solo paso como un bloque "try" pseudo
- Ken Worthley, por sus ideas sobre la implementación original de clew, y por escribir la función que ahora se llama error.InSubscriptRethrow
- Anders Monsen, por idear el nombre "clew", y por ser uno de los primeros en adoptar la técnica
- Mis otros varios colegas de Soliant, quienes proporcionaron valiosas ideas y retroalimentación que ayudaron a que clew sea lo que es hoy
