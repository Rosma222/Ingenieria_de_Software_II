**2026/06/12 - ACTIVIDAD ASINCRÓNICA**
*Basados en el modelo componentes adjunto y que venimos trabajando y basados en la documentación asociada. Escribir un README.md con formato Markdown (a modo de listado de requerimientos) de un posible y potencial componente que ajuste al modelo de componentes y que además sea un módulo con la suficiente independencia para justificar ser un componente. En conclusión un nuevo ejemplo de componente. Posteriormente analizaremos en la próxima clase la viabilidad y factibilidad de dicho componente. Adjunto documentación oficial de la sintaxis de Markdown y el repositorio y la documentación del misma ya compartida en otras actividades.*


# Componente Validador de Correo Electrónico (EmailValidator)

## Descripción General

El componente **EmailValidator** tiene como responsabilidad exclusiva validar direcciones de correo electrónico según reglas sintácticas predefinidas.

Su objetivo es ofrecer un servicio reutilizable y desacoplado para cualquier aplicación que requiera verificar la validez de una dirección de correo electrónico antes de almacenarla o procesarla.

Este componente se ajusta al modelo de componentes propuesto, pudiendo ser cargado dinámicamente mediante el `ModuleManager` y utilizado a través de una interfaz contractual sin necesidad de conocer su implementación interna.

---

# Justificación como Componente

La validación de correos electrónicos constituye una funcionalidad transversal utilizada por múltiples sistemas:

* Sistemas de registro de usuarios.
* Aplicaciones de gestión empresarial.
* Sistemas de autenticación.
* Plataformas de comercio electrónico.
* Aplicaciones web y móviles.

La lógica de validación puede evolucionar independientemente del sistema anfitrión, permitiendo:

* Reutilización.
* Sustitución transparente de implementaciones.
* Mantenimiento independiente.
* Actualización.

Por estas razones se considera una funcionalidad con suficiente independencia funcional para justificar su implementación como componente.

---

# Responsabilidad Única

El componente posee una única responsabilidad:

> Determinar si una dirección de correo electrónico cumple con las reglas de validación establecidas.

No administra usuarios.

No almacena información.

No realiza conexiones de red.

No envía correos electrónicos.

No interactúa con bases de datos.

---

# Interfaz Propuesta

Archivo:

```cpp
i_email_validator.hpp
```

```cpp
#ifndef IEMAILVALIDATOR_HPP
#define IEMAILVALIDATOR_HPP

#include "i_component.hpp"

class IEmailValidator : public IComponent
{
public:

    virtual ComponentResult validate_email(
        const char* email, 
        bool* result
    ) noexcept = 0;

};

#endif // IEMAILVALIDATOR_HPP
```

---

# Operaciones Disponibles

## validate_email

### Descripción

Valida una dirección de correo electrónico recibida como parámetro.

### Parámetros

| Parámetro | Descripción                                       |
| --------- | ------------------------------------------------- |
| email     | Dirección de correo electrónico a validar         |
| result    | Variable de salida donde se almacena el resultado |

### Retorno

| Código                 | Significado                       |
| ---------------------- | --------------------------------- |
| SUCCESS                | Operación realizada correctamente |
| ERROR_INVALID_ARGUMENT | Parámetros inválidos              |
| ERROR_INTERNAL         | Error interno del componente      |

### Resultado

| Valor de result | Significado     |
| --------------- | --------------- |
| true            | Correo válido   |
| false           | Correo inválido |

---

# Reglas de Validación

La implementación inicial deberá verificar:

* Existencia del carácter '@'.
* Existencia de un dominio posterior al '@'.
* Existencia de al menos un punto '.' dentro del dominio.
* Longitud mínima de la dirección.
* Ausencia de espacios en blanco.

Ejemplos válidos:

```text
usuario@gmail.com
nombre.apellido@empresa.com
soporte@universidad.edu.ar
```

Ejemplos inválidos:

```text
usuariogmail.com
usuario@
@gmail.com
usuario @gmail.com
usuario@gmail
```

---

# Dependencias

El componente no requiere dependencias externas.

Puede implementarse utilizando exclusivamente la Biblioteca Estándar de C++.

---

# Integración con el Modelo de Componentes

## Qué es extern "C"?
* Cuando se compila en C++, el compilador modifica los nombres de las funciones internamente.
Usando `extern C` el compilador NO modifica el nombre. Permite que el Host encuentre funciones exportadas por nombre.

### El componente deberá exportar las funciones estándar definidas por el modelo:

* Validar la versión del ABI.
```cpp
extern "C" int get_api_version() noexcept;
```

* Crear instancias del componente.
```cpp
extern "C" IComponent* create_component() noexcept;
```
Es una función fábrica exportada por la DLL que crea una instancia del componente y la devuelve como `IComponent*`.

* Destruir instancias de forma segura.
```cpp
extern "C" void destroy_component(IComponent* instance) noexcept;
```
La DLL destruye el objeto que ella misma creó y libera la memoria que retuvo.


---

# Ejemplo de Uso

```cpp
ModuleManager manager; //Se crea el administrador de módulos p/ cargar DLLs y crear instancias

manager.load_module("./lib/EmailValidator"); //Carga la libreria EmailValidator. Internamente ModuleManager:
                                            // 1) Busca la DLL llamada "EmailValidator".
                                            // 2) Obtiene get_api_version.
                                            // 3) Obtiene create_component.
                                            // 4) Obtiene destroy_component.
                                            // 5) Verifica compatibilidad de ABI.
                                            // 6) Llama a create_component().
                                            // 7) Hace dynamic_cast<IEmailValidator*>.
                                            // 8) Devuelve un shared_ptr<IEmailValidator>.
auto validator =   //Solicita una instancia del componente
manager.create_instance<IEmailValidator>("EmailValidator");

bool is_valid = false; //Variable donde el componente escribirá el resultado.
                       // false = correo inválido - true  = correo válido

ComponentResult result =   //Se invoca el método 
validator->validate_email(
    "usuario@gmail.com",   // correo a validar
    &is_valid              // SUCCESS = true  o false
);

if(result == ComponentResult::SUCCESS && is_valid)  // Verifica dos condiciones:
                                                    // 1) Que la operación se ejecutó correctamente.
                                                    // 2) Que el correo fue considerado válido
{
    std::cout << "Correo válido";
}
```
---

# Secuencia

El Host carga dinámicamente la biblioteca `EmailValidator` mediante `ModuleManager`, crea una instancia a través de la función exportada `create_component()`, utiliza el componente mediante la interfaz `IEmailValidator` y finalmente la instancia es destruida automáticamente mediante `destroy_component()` cuando el puntero libera el recurso.

---

# Beneficios

* Reutilización entre proyectos.
* Independencia de implementación.
* Compatibilidad con carga dinámica.
* Cumplimiento del Principio de Responsabilidad Única.
* Posibilidad de incorporar nuevas reglas sin modificar el Host.
* Sustitución transparente por futuras versiones.

---

# Posibles refactoring

Versiones posteriores podrían incorporar:

* Verificación de dominios permitidos.
* Verificación de dominios bloqueados.
* Validación internacionalizada de direcciones de correo electrónico.

---

# Conclusión

El componente EmailValidator representa una funcionalidad reutilizable, desacoplada y de propósito específico que puede ser integrada en múltiples aplicaciones sin depender de su lógica de negocio interna.

Su diseño cumple con los principios del modelo de componentes presentado en el ejemplo `Greeter` , permitiendo composición independiente, encapsulación de implementación y reutilización mediante interfaces contractuales.
