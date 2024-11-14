## Reflexion sobre el proceso de diseño de una API

- **Definicon de estructrura**: Cuando se cre la estrtuctura de datos nos debemos asegurar de que la estructura sea lo suficientemente flexible para adaptarse a posibles cambios futuros, a su vez estas deben ser intuitivas y escalables sin perder  claridad.
- **Enfoque RESTful**: Se debe tener en cuenta que la API sea RESTful, es decir, que siga los principios de REST, como la utilizacion de metodos HTTP, URIs y codigos de estado.
- - Cada verbo tiene un semantica bien efinida que permite a otros desarrolladores entender rapidamente que tipo de operación se esta realizando:
- - Usar un verbo que no corrersponde con la operacion que se esta realizando puede tener consecuencias negativas en terminos de seguridad, consistencia, mantenibilidad y escalabilidad.
- Un concepto clave al diseñar una API es la **consistencia**. La consistencia en una API se refiere a la forma en que se estructuran las solicitudes y respuestas, y cómo se nombran los recursos. La consistencia es importante porque permite a los desarrolladores predecir cómo se comportará la API en diferentes situaciones.
- La **idenpotencia** es otro concepto clabe, este se refiere a la capacidad de una operación para ejecutarse multiples veces sin cambiar el resultado mas alla de su primer efecto. Esto otras palabras significa que una operacion oriducura el mismo efecto en el servidor sin imporar cuantas veces sea ejecutada.

Tomando en cuenta la anterior es importante tener en cuenta que al ser un estandar, da un marco de referencia que puede ser entendido por todo aquel que conozca el estandar, lo que facilita la comprension y el mantenimiento de la API.
