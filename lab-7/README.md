# 1.- Design Problem Solving:
## Scenario Description: 

**Participants are provided with a series of common software design challenges. They will need to choose appropriate design patterns to solve these specific problems effectively.**

## Design Challenges:

**Global Configuration Management:** Design a system that ensures a single, globally accessible configuration object without access conflicts.

- En este ejeplo se muestra un patron Singleton para la creacion de un objeto Logger. El objetivo del singleton es asegurarse de que solo exista una instancia del objetos por todo el ciclo de vida del programa. De esta forma todo el logueo del aplicativo estara centralizado.

```java
public class Logger {

    private static volatile Logger instance;


    private Logger() {

        System.out.println("Logger inicializado.");
    }


    public static Logger getInstance() {
        if (instance == null) { 
            synchronized (Logger.class) {
                if (instance == null) { 
                    instance = new Logger();
                }
            }
        }
        return instance;
    }

    public void log(String message) {
        System.out.println("LOG: " + message);
    }
}
```
- La implementación del código anterior sería de la siguiente forma.

```java
public class Application {
    public static void main(String[] args) {

        Logger logger = Logger.getInstance();
        
        logger.log("Aplicación iniciada.");
        logger.log("Realizando alguna operación importante.");
        logger.log("Aplicación finalizada.");
    }
}
```

**Dynamic Object Creation Based on User Input:** Implement a system to dynamically create various types of user interface elements based on user actions.

- Para el siguiente ejemplo se creará un patron "abstract methond", de esta manera tendremos una gerarquia de clases de las cuales se delagá la cración de la instancia especifica a una clase base na idea es no especificar explicitamente el tipo exacto del objeto que se creará.

Supongamos la creación de repostes del cual se puede generar en formato PDF o CSV o excel.

```java
public abstract class Report {
    public abstract void generate();
}
```
Creamos las diferentes clases que crearan el reporte en el formato deseado.

```java
public class PdfReport extends Report {
    @Override
    public void generate() {
        System.out.println("Generando reporte en formato PDF...");
    }
}

public class CsvReport extends Report {
    @Override
    public void generate() {
        System.out.println("Generando reporte en formato CSV...");
    }
}

public class ExcelReport extends Report {
    @Override
    public void generate() {
        System.out.println("Generando reporte en formato Excel...");
    }
}
```

Creamos una clase abstracta que sea el "Factory" de los reportes.

```java
public class ReportFactory {
    public static Report createReport(ReportType type) {
        switch (type) {
            case PDF:
                return new PdfReport();
            case CSV:
                return new CsvReport();
            case EXCEL:
                return new ExcelReport();
            default:
                throw new IllegalArgumentException("Tipo de reporte no soportado");
        }
    }
}
```

La implementación del código anterior sería de la siguiente forma.

```java
public class Application {
    public static void main(String[] args) {
        Report pdfReport = ReportFactory.createReport(ReportType.PDF);
        pdfReport.generate();

        Report csvReport = ReportFactory.createReport(ReportType.CSV);
        csvReport.generate();

        Report excelReport = ReportFactory.createReport(ReportType.EXCEL);
        excelReport.generate();
    }
}
```

**State Change Notification Across System Components:** Ensure components are notified about changes in the state of other parts without creating tight coupling.

- Para este caso se hará uso del patron Observer, el cual permite que un objeto notifique a otros objetos sobre cambios en su estado.

- Como caso de uso tenemos un sistema de notificaciones de temperatura, suponiendo que se necesita saber si la temperatura de una abitación sube o baja y notificar a otros sistemas para realizar diferentes acciones. Para esto se creara un sistema de alertas y una pantalla de visualizacion, que responsan al cambio de temperatura que proporciona un sensor.

Se crea ininterfaz observer que los observadores implementarán.

```java
public interface Observer {
    void update(float temperature);
}
```
Se crea una clase que será el sunject que permitira a los observadores, suscribirser, cancelar subscripcion y recibir notificaciones.

```java
public interface Subject {
    void addObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers();
}
```

Se crea una clase que implementa el subject y notifica a los observadores.

```java
public class TemperatureSensor implements Subject {
    private List<Observer> observers;
    private float temperature;

    public TemperatureSensor() {
        this.observers = new ArrayList<>();
    }

    @Override
    public void addObserver(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(temperature);
        }
    }

    // Método para actualizar la temperatura y notificar a los observadores
    public void setTemperature(float temperature) {
        this.temperature = temperature;
        notifyObservers();
    }
}
```

Ahora la clase AlertSystem que implementa observer usará el metodo update para dar su respuesta al cambio de temperatura. Este es un observador.

```java
public class AlertSystem implements Observer {
    @Override
    public void update(float temperature) {
        if (temperature > 30) {
            System.out.println("¡Alerta! La temperatura está demasiado alta: " + temperature + "°C");
        } else if (temperature < 15) {
            System.out.println("¡Alerta! La temperatura está demasiado baja: " + temperature + "°C");
        }
    }
}
```

Ejemplo de implementación.
    
```java
    public static void main(String[] args) {
        // Crear el sujeto
        TemperatureSensor sensor = new TemperatureSensor();

        // Crear los observadores
        AlertSystem alertSystem = new AlertSystem();
        DisplayScreen displayScreen = new DisplayScreen();

        // Añadir los observadores al sensor
        sensor.addObserver(alertSystem);
        sensor.addObserver(displayScreen);

        // Cambiar la temperatura y notificar a los observadores
        System.out.println("Cambiando la temperatura a 20°C...");
        sensor.setTemperature(20);

        System.out.println("Cambiando la temperatura a 32°C...");
        sensor.setTemperature(32);

        System.out.println("Cambiando la temperatura a 10°C...");
        sensor.setTemperature(10);
    }
```


**Efficient Management of Asynchronous Operations:** Manage multiple asynchronous operations like API calls which need to be coordinated without blocking the main application workflow.

- Para este caso podemos hacer uso del patron mediador.
- En este caso de uso tenemos un chat grupal, donde los usuarios pueden enviar mensajes a todos los usuarios del grupo. Para esto se creara un sistema de chat donde los usuarios enviaran mensajes y el mediador se encargara de enviar los mensajes a todos los usuarios del grupo.
- El patron mediador en este caso nos ayudara a tener una comunicacion vilateral entre los usuarios y el mediador, de esta forma los usuarios no se comunican directamente entre ellos.

Se crea una interfaz que será el mediador.

```java
public interface ChatMediator {
    void sendMessage(String message, User user);
    void addUser(User user);
}
```
Se crea la clase user que sara el mediador.
    
```java
public abstract class User {
    protected ChatMediator mediator;
    protected String name;

    public User(ChatMediator mediator, String name) {
        this.mediator = mediator;
        this.name = name;
        
        mediator.addUser(this); // Agregar el usuario al mediador
    }

    public abstract void send(String message);
    public abstract void receive(String message);
}
```

En esta clase implementaremos la funcionalidad concreta de enviar y recibir mensajes.

```java
public class UserImpl extends User {

    public UserImpl(ChatMediator mediator, String name) {
        super(mediator, name);
    }

    @Override
    public void send(String message) {
        System.out.println(this.name + " envía: " + message);
        mediator.sendMessage(message, this);
    }

    @Override
    public void receive(String message) {
        System.out.println(this.name + " recibe: " + message);
    }
}
```

La siguiente clase se encarga de la funcionalidad concreta del mediador y distribuir los mensajes entre los susuarios

```java
public class ChatRoomMediator implements ChatMediator {
    private List<User> users;

    public ChatRoomMediator() {
        this.users = new ArrayList<>();
    }

    @Override
    public void addUser(User user) {
        users.add(user);
    }

    @Override
    public void sendMessage(String message, User user) {
        for (User u : users) {
            // Enviar el mensaje a todos menos al remitente
            if (u != user) {
                u.receive(message);
            }
        }
    }
}
```
Implementación del código anterior sería de la siguiente forma.

```java
    public static void main(String[] args) {
        ChatMediator mediator = new ChatRoomMediator();

        User user1 = new UserImpl(mediator, "Paco");
        User user2 = new UserImpl(mediator, "Carlos");
        User user3 = new UserImpl(mediator, "Maria");

        // Enviar mensaje desde un usuario
        user1.send("Hola a todos!");
    }
```

El caso anterior tambien se puede abordar usando un CompletableFeature de la siguiente forma.

En este caso el metodo de sendMessage se modifica para que sea asíncrono, de esta forma se evita bloquear el flujo principal de la aplicación.

```java
public class ChatRoomMediator implements ChatMediator {
    private List<User> users = new ArrayList<>();

    @Override
    public void addUser(User user) {
        users.add(user);
    }

    // Método asíncrono para enviar mensajes
    @Override
    public void sendMessage(String message, User sender) {
        for (User user : users) {
            if (user != sender) {
                // Enviar el mensaje de forma asíncrona a cada usuario
                CompletableFuture.runAsync(() -> user.receive(message));
            }
        }
    }
}
```

De igual forma el envio de mensajes del usuario se modifica para que sea asíncrono.

```java
public abstract class User {
    protected ChatMediator mediator;
    protected String name;

    public User(ChatMediator mediator, String name) {
        this.mediator = mediator;
        this.name = name;
        mediator.addUser(this);
    }

    public abstract void send(String message);

    public abstract void receive(String message);
}
```

```java
public class UserImpl extends User {

    public UserImpl(ChatMediator mediator, String name) {
        super(mediator, name);
    }

    @Override
    public void send(String message) {
        System.out.println(this.name + " envía: " + message);
        // Llamada asíncrona al mediador
        CompletableFuture.runAsync(() -> mediator.sendMessage(message, this));
    }

    @Override
    public void receive(String message) {
        System.out.println(this.name + " recibe: " + message);
    }
}
```

En este caso descrito anteriormente el mediador se encarga de que los usuarios recivan el mensaje de forma asíncrona en un hilo serparado del principal



**Task:** Outline solutions that integrate these patterns into a cohesive design to address the challenges.