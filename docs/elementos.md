# Documentación y prueba del programa.
*Elementos de Python*

1. [Introducción.](#introduccion)
2. [Aplicación.](#aplicacion)
3. [Implementación de los comentarios.](#comentarios-en-el-codigo)
    4. [Fichero lavadero.py](#lavadero)
    5. [Fichero main_app.py](#main_app)

---

## Introducción.

Analizaremos el código de la aplicación lavadero, la cual se compone de dos ficheros:

- `lavadero.py`
- `main_app.py`

## Aplicación.

La aplicación modela el ciclo de vida completo del proceso de limpieza de un vehículo, funcionando como una máquina de estados que gestiona las transiciones entre las distintas etapas del lavado según los servicios contratados.

## Comentarios en el código.

### lavadero

El fichero `lavadero.py` contiene la clase **Lavadero**, la cual deglosaremos en los siguientes apartados:

1. [Clase](#clase)
2. [Getters de la clase](#getter-de-la-clase)
3. [Función terminar](#metodo-terminar)
4. [Función hacerLavado](#metodo-hacerlavado)
5. [La función cobrar](#metodo-cobrar)
6. [La función avanzarFase](#metodo-avanzarfase)
7. [La función imprimir_fase](#metodo-imprimir_fase)
8. [La función imprimir_estado](#metodo-imprimir_estado)
9. [La función ejecutar_y_obtener_fases](#metodo-ejecutar_y_obtener_fases)


<a id="clase"></a>
### Comenzamos con la clase **Lavadero** y el constructor.

```python
class Lavadero:
    """
    Simula el estado y las operaciones de un túnel de lavado de coches.
    Cumple con los requisitos de estado, avance de fase y reglas de negocio.
    """

    FASE_INACTIVO = 0
    FASE_COBRANDO = 1
    FASE_PRELAVADO_MANO = 2
    FASE_ECHANDO_AGUA = 3
    FASE_ENJABONANDO = 4
    FASE_RODILLOS = 5
    FASE_SECADO_AUTOMATICO = 6
    FASE_SECADO_MANO = 7
    FASE_ENCERADO = 8

    def __init__(self):
        """
        Constructor de la clase Lavadero.
        Inicializa los atributos privados relacionados con los ingresos, la fase actual
        y el estado inicial y opciones de lavado.
        """
        self.__ingresos = 0.0  # Acumulado de ingresos totales.
        self.__fase = self.FASE_INACTIVO  # Estado inicial del lavadero.
        self.__ocupado = False  # Indica si hay un coche siendo atendido.
        self.__prelavado_a_mano = False  # Opción de prelavado manual.
        self.__secado_a_mano = False  # Opción de secado manual.
        self.__encerado = False  # Opción de encerado.
        self.terminar()  # Asegura que el estado inicial sea consistente.
```

<a id="getter-de-la-clase"></a>
### Aquí se definirán dichas propedades o getters de la clase, que permitirán leer atributos privados de la clase como si fueran variables normales, protegiendo los datos originales.

```Python
    @property
    def fase(self):
        """Devuelve la fase actual del proceso de lavado."""
        return self.__fase

    @property
    def ingresos(self):
        """Devuelve el total de ingresos acumulados del lavadero."""
        return self.__ingresos

    @property
    def ocupado(self):
        """Devuelve True si el lavadero está actualmente ocupado, 
        False en caso contrario."""
        return self.__ocupado

    @property
    def prelavado_a_mano(self):
        """Devuelve True si se ha seleccionado la opción de prelavado a mano."""
        return self.__prelavado_a_mano

    @property
    def secado_a_mano(self):
        """Devuelve True si se ha seleccionado la opción de secado a mano."""
        return self.__secado_a_mano

    @property
    def encerado(self):
        """Devuelve True si se ha seleccionado la opción de encerado."""
        return self.__encerado
```

<a id="metodo-terminar"></a>
### Método **terminar** cuyo objetivo principal es limpiar toda la configuración actual después de que un proceso de lavado ha concluido o ha sido cancelado.

```Python
    def terminar(self):
        """
        Restablece el lavadero a su estado inicial de inactividad.
        Limpia las opciones seleccionadas y marca el lavadero como libre.
        """
        self.__fase = self.FASE_INACTIVO
        self.__ocupado = False
        self.__prelavado_a_mano = False
        self.__secado_a_mano = False
        self.__encerado = False
```

<a id="metodo-hacerlavado"></a>
### Método **hacerLavado**, es el punto de entrada para iniciar un servicio de lavado. Su trabajo es validar que el pedido sea correcto y configurar el lavadero para empezar a funcionar.

```Python
    def hacerLavado(self, prelavado_a_mano, secado_a_mano, encerado):
        """
        Inicia un nuevo ciclo de lavado con las opciones especificadas.

        Argumentos:
            prelavado_a_mano (bool): Indica si se requiere prelavado manual.
            secado_a_mano (bool): Indica si se requiere secado manual.
            encerado (bool): Indica si se requiere encerado.

        Manejador de errores:
            RuntimeError: Si el lavadero ya está ocupado.
            ValueError: Si se solicita encerado sin secado a mano, 
            el programa impide realizarlo así.
        """
        if self.__ocupado:
            raise RuntimeError(
                "No se puede iniciar un nuevo lavado mientras el lavadero está ocupado"
            )

        if not secado_a_mano and encerado:
            raise ValueError("No se puede encerar el coche sin secado a mano")

        self.__fase = self.FASE_INACTIVO
        self.__ocupado = True
        self.__prelavado_a_mano = prelavado_a_mano
        self.__secado_a_mano = secado_a_mano
        self.__encerado = encerado
```

<a id="metodo-cobrar"></a>
### Método **cobrar**, es el módulo de facturación del lavadero. Su trabajo es calcular cuánto debe pagar el cliente y sumar ese dinero a la caja total del lavadero.

```Python
    def _cobrar(self):
        """
        Calcula el coste total del lavado en función de los servicios 
        adicionales seleccionados.
        Actualiza el total de ingresos del lavadero.

        Returns:
            float: El precio calculado del lavado actual.
        """
        coste_lavado = 5.00  # Precio base del lavado.

        if self.__prelavado_a_mano:
            coste_lavado += 1.50  # Suplemento por prelavado manual.

        if self.__secado_a_mano:
            coste_lavado += 1.20  # Suplemento por secado manual.

        if self.__encerado:
            coste_lavado += 1.00  # Suplemento por encerado.

        self.__ingresos += coste_lavado
        return coste_lavado
```

<a id="metodo-avanzarfase"></a>
### Método **avanzarFase**, es el cerebro lógico o el *controlador* del lavadero.

```Python
    def avanzarFase(self):
        """
        Avanza el estado del lavadero a la siguiente fase lógica.
        Maneja las transiciones de estados y realiza acciones como cobrar o finalizar.
        """
        if not self.__ocupado:
            return  # Si no hay coche, no hacemos nada.

        if self.__fase == self.FASE_INACTIVO:
            # Al iniciar, cobramos y pasamos a estado COBRANDO.
            coste_cobrado = self._cobrar()
            self.__fase = self.FASE_COBRANDO
            print(f" (COBRADO: {coste_cobrado:.2f} €) ", end="")

        elif self.__fase == self.FASE_COBRANDO:
            # Decidimos si vamos a prelavado manual o directo al agua.
            if self.__prelavado_a_mano:
                self.__fase = self.FASE_PRELAVADO_MANO
            else:
                self.__fase = self.FASE_ECHANDO_AGUA

        elif self.__fase == self.FASE_PRELAVADO_MANO:
            # Del prelavado manual pasamos a echar agua.
            self.__fase = self.FASE_ECHANDO_AGUA

        elif self.__fase == self.FASE_ECHANDO_AGUA:
            # De echar agua pasamos a enjabonar.
            self.__fase = self.FASE_ENJABONANDO

        elif self.__fase == self.FASE_ENJABONANDO:
            # De enjabonar pasamos a los rodillos.
            self.__fase = self.FASE_RODILLOS

        elif self.__fase == self.FASE_RODILLOS:
            # Tras los rodillos, decidimos tipo de secado.
            if self.__secado_a_mano:
                self.__fase = self.FASE_SECADO_AUTOMATICO
            else:
                self.__fase = self.FASE_SECADO_MANO

        elif self.__fase == self.FASE_SECADO_AUTOMATICO:
            # Fin tras secado automático.
            self.terminar()

        elif self.__fase == self.FASE_SECADO_MANO:
            # Fin tras secado a mano.
            self.terminar()

        elif self.__fase == self.FASE_ENCERADO:
            # Fin tras encerado.
            self.terminar()
            
        """    
        Si por algún error de programación 
        el lavadero entra en una fase que no existe.
        """
        else:
            raise RuntimeError(
                f"Estado no válido: Fase {self.__fase}. El lavadero va a estallar..."
            )

```

<a id="metodo-imprimir_fase"></a>
### Método **imprimir_fase**, es la encargada de la interfaz de usuario, permitiendo que cualquier persona sepa exactamente qué está pasando en el lavadero en cada momento.

```Python
    def imprimir_fase(self):
        """
        Imprime por pantalla la descripción textual de la fase actual.
        Utiliza un diccionario para mapear los códigos de fase a texto.
        """
        fases_map = {
            self.FASE_INACTIVO: "0 - Inactivo",
            self.FASE_COBRANDO: "1 - Cobrando",
            self.FASE_PRELAVADO_MANO: "2 - Haciendo prelavado a mano",
            self.FASE_ECHANDO_AGUA: "3 - Echándole agua",
            self.FASE_ENJABONANDO: "4 - Enjabonando",
            self.FASE_RODILLOS: "5 - Pasando rodillos",
            self.FASE_SECADO_AUTOMATICO: "6 - Haciendo secado automático",
            self.FASE_SECADO_MANO: "7 - Haciendo secado a mano",
            self.FASE_ENCERADO: "8 - Encerando a mano",
        }
        print(
            fases_map.get(self.__fase, f"{self.__fase} - En estado no válido"), end=""
        )
```

<a id="metodo-imprimir_estado"></a>
### Método **imprimir_estado**, es el generador de informes o el *panel de control* visual de la aplicación. Su trabajo es mostrarle al usuario (o al administrador del lavadero) una "foto" exacta de qué está pasando en ese preciso momento.

```Python
    def imprimir_estado(self):
        """
        Imprime un informe completo del estado actual del lavadero,
        incluyendo ingresos, ocupación, configuración de lavado y fase actual.
        """
        print("----------------------------------------")
        print(f"Ingresos Acumulados: {self.ingresos:.2f} €")
        print(f"Ocupado: {self.ocupado}")
        print(f"Prelavado a mano: {self.prelavado_a_mano}")
        print(f"Secado a mano: {self.secado_a_mano}")
        print(f"Encerado: {self.encerado}")
        print("Fase: ", end="")
        self.imprimir_fase()
        print("\n----------------------------------------")
```

<a id="metodo-ejecutar_y_obtener_fases"></a>
### Método **ejecutar_y_obtener_fases**, es el simulador automático del sistema. Su trabajo es ejecutar un proceso de lavado completo de principio a fin y registrar todo lo que sucede en una lista.

```Python
def ejecutar_y_obtener_fases(self, prelavado, secado, encerado):
    """
    Ejecuta un ciclo completo y devuelve la lista de fases visitadas.
    Útil para pruebas o simulación.

    Argumetnos:
        self (Lavadero): Instancia del lavadero a controlar.
        prelavado (bool): Opción de prelavado.
        secado (bool): Opción de secado.
        encerado (bool): Opción de encerado.

    Returns:
        list: Lista de fases por las que ha pasado el lavadero.
    """
    self._hacer_lavado(
        prelavado, secado, encerado
    )
    fases_visitadas = [self.fase]

    while self.ocupado:
        # Usamos un límite de pasos para evitar bucles infinitos 
        # en caso de error o configuración cíclica
        if len(fases_visitadas) > 15:
            raise Exception("Bucle infinito detectado en la simulación de fases.")
        self.avanzarFase()  # Avanza un paso en la máquina de estados
        fases_visitadas.append(self.fase)  # Registra la nueva fase

    return fases_visitadas
```

### main_app

Con fichero `main_app.py` iniciaremos una instancia para probar la clase lavadero explicada anteriormente.

1. [Importación de la clase Lavadero](#importacion-de-la-clase-lavadero)
2. [Inicialización de la aplicación](#inicializacion-de-la-aplicacion)

<a id="importacion-de-la-clase-lavadero"></a>
### Importamos la clase lavadero para utilizarla.

```Python
# main_app.py

# Importar la clase desde el otro archivo (módulo)
from lavadero import Lavadero
```

<a id="inicializacion-de-la-aplicacion"></a>
### Inicializamos la aplicación lavadero.

```Python
# MODIFICACIÓN CLAVE AQUÍ: La función ahora acepta 3 argumentos
def ejecutarSimulacion(lavadero, prelavado, secado_mano, encerado):
    """
    Simula el proceso de lavado para un vehículo con las opciones dadas.
    Ahora acepta una instancia de lavadero.

    :param lavadero: Instancia de Lavadero.
    :param prelavado: bool, True si se solicita prelavado a mano.
    :param secado_mano: bool, True si se solicita secado a mano.
    :param encerado: bool, True si se solicita encerado.
    """
    
    print("--- INICIO: Prueba de Lavado con Opciones Personalizadas ---")
```

```Python
# Mostrar las opciones solicitadas
    print(f"Opciones solicitadas: [Prelavado: {prelavado}, Secado a mano: {secado_mano}, Encerado: {encerado}]")
```

```Python
    # 1. Iniciar el lavado
    try:
        # Esto establece las opciones y pasa a Fase 0 (Inactivo, pero Ocupado=True)
        lavadero.hacerLavado(prelavado, secado_mano, encerado)
        print("\nCoche entra. Estado inicial:")
        lavadero.imprimir_estado()

        # 2. Avanza por las fases
        print("\nAVANZANDO FASE POR FASE:")
        
        # Usamos un contador para evitar bucles infinitos en caso de error o bucles inesperados
        pasos = 0
        while lavadero.ocupado and pasos < 20: 
            # El cobro ahora ocurre en la primera llamada a avanzarFase (transición 0 -> 1)
            lavadero.avanzarFase()
            print(f"-> Fase actual: ", end="")
            lavadero.imprimir_fase()
            print() 
            pasos += 1
        
        print("\n----------------------------------------")
        print("Lavado completo. Estado final:")
        lavadero.imprimir_estado()
        print(f"Ingresos acumulados: {lavadero.ingresos:.2f} €")
        print("----------------------------------------")
        
    except ValueError as e: # Captura la excepción de regla de negocio (Requisito 2)
        print(f"ERROR DE ARGUMENTO: {e}")
    except RuntimeError as e: # Captura la excepción de estado (Requisito 3)
        print(f"ERROR DE ESTADO: {e}")
    except Exception as e:
        print(f"ERROR INESPERADO: {e}")
```

```Python
# Punto de entrada (main): Aquí pasamos los parámetros
if __name__ == "__main__":
    
    lavadero_global = Lavadero() # Usamos una única instancia para acumular ingresos
    
    # EJEMPLO 1: Lavado completo con prelavado, secado a mano, con encerado (Requisito 8 y 14)
    # Precio esperado: 5.00 + 1.50 + 1.00 + 1.20 = 8.70 €
    print("\n=======================================================")
    print("EJEMPLO 1: Prelavado (S), Secado a mano (S), Encerado (S)")
    ejecutarSimulacion(lavadero_global, prelavado=True, secado_mano=True, encerado=True)
    
    # EJEMPLO 2: Lavado rápido sin extras (Requisito 9)
    # Precio esperado: 5.00 €
    print("\n=======================================================")
    print("EJEMPLO 2: Sin extras (Prelavado: N, Secado a mano: N, Encerado: N)")
    ejecutarSimulacion(lavadero_global, prelavado=False, secado_mano=False, encerado=False)

    # EJEMPLO 3: Lavado con encerado, pero sin secado a mano (Debe lanzar ValueError - Requisito 2)
    print("\n=======================================================")
    print("EJEMPLO 3: ERROR (Encerado S, Secado a mano N)")
    ejecutarSimulacion(lavadero_global, prelavado=False, secado_mano=False, encerado=True)

    # EJEMPLO 4: Lavado con prelavado a mano (Requisito 4 y 10)
    # Precio esperado: 5.00 + 1.50 = 6.50 €
    print("\n=======================================================")
    print("EJEMPLO 4: Prelavado (S), Secado a mano (N), Encerado (N)")
    ejecutarSimulacion(lavadero_global, prelavado=True, secado_mano=False)
```
