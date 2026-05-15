                        Caso 1 — Archivo: AnimalService.php:
Ficha de análisis:

1. ¿Cuántos 'actores' distintos tienen razones para pedir cambios a esta clase?
3 actores:
El equipo de negocio (reglas de validación del animal)
El equipo de comunicaciones (plantilla y lógica del email)
El equipo de reportes/documentos (formato y generación del PDF)

2. Nombre los ejes de cambio (cada razón independiente para cambiar):
Eje 1: Lógica de negocio (validaciones y persistencia del animal)
Eje 2: Notificación por email (plantilla HTML, destinatarios, asunto)
Eje 3: Generación de PDF (vista, ruta de almacenamiento, librería)

3. Si tuviéramos que escribir una prueba unitaria para la validación del arete, ¿qué
dependencias habría que mockear?
Animal::create() (Eloquent / base de datos)
DB::table('propietarios') (consulta a BD)
Mail::raw() (servicio de email)
Pdf::loadView() y $pdf->save() (generación y escritura de archivo)





                        Caso 2 — Archivo: ReporteService.php

1. ¿Cuántos archivos habría que modificar para agregar el tipo 'resumen_mensual'?
Al menos 1 archivo obligatorio: ReporteService.php (abrir el switch y agregar el nuevo case). Pero esto implica abrir una clase que ya está probada

2. ¿Qué abstracción (interfaz o clase base) permitiría agregar nuevos tipos sin tocar
ReporteService?
Una interfaz IGeneradorReporte con un método generar(int $ranchoId).

3. ¿Cómo debería ReporteService recibir el generador correcto? (constructor injection,
factory, service locator...)
Mediante un Factory (ReporteFactory) que mapea el string del tipo a la clase concreta correspondiente, y luego se inyecta en ReporteService vía constructor injection.



                               Caso 3 — Archivo: AnimalSinPeso.php

1. ¿Cuál es el contrato que Animal promete con agregarRegistroPeso()?
Que cualquier objeto de tipo Animal puede recibir un RegistroPeso y procesarlo sin lanzar excepción. El cliente confía en que la operación se ejecutará correctamente para cualquier instancia que se declare como Animal.

2. Si el problema es que AnimalSinPeso no puede tener registros de peso, ¿debería heredar
de Animal? ¿Cuál es la alternativa de diseño?
No. La herencia aquí es incorrecta porque AnimalSinPeso no cumple el comportamiento completo de Animal. La alternativa es separar la capacidad de pesaje en una interfaz, de modo que solo las clases que realmente soportan registros de peso la implementen.

3. Proponga una jerarquía de clases alternativa que no viole LSP (puede ser una interfaz o
una clase diferente).
Extraer la capacidad de pesaje a una interfaz IPesable, que solo implementan los animales que realmente admiten registros de peso. AnimalSinPeso y Animal comparten datos base mediante una clase abstracta AnimalBase, pero solo Animal es IPesable.

                             Caso 4 — Archivo: IGestorAnimal.php

1. Cuente los métodos de IGestorAnimal que ReportadorSoloLectura implementa con stubs.
¿Cuántos?
5 métodos:
crear, actualizar, eliminar, agregarRegistroPeso y asignarFotografia.

2. Si añadimos un método actualizarRaza(int $id, int $razaId) a IGestorAnimal, ¿cuántos
archivos habría que tocar?
Todos los implementadores de IGestorAnimal, incluyendo ReportadorSoloLectura que nunca usaría ese método.

3. ¿En cuántas interfaces más cohesivas descompondría IGestorAnimal? Proponga los
nombres.

En 3 interfaces cohesivas:
ILectorAnimal — operaciones de lectura 
IEscritorAnimal — operaciones de escritura
IGestorAnimal — extiende ambas, para quien necesite todo

                            Caso 5 — Archivo: EstimadorPesoService.php

1. Identifique el módulo de alto nivel y el módulo de bajo nivel en este código.
Alto nivel: EstimadorPesoService — orquesta la lógica de negocio.
Bajo nivel: el cliente HTTP de Laravel + la URL hardcodeada de Flask/Python.

2. ¿Cuántos pasos se necesitarían para escribir un test unitario de estimar() sin levantar
Flask?
Con el código actual es muy difícil porque la URL y el cliente están "hardcoded". Habría que usar técnicas complejas de interceptación de red o "Fakes" del Framework en lugar de una simple sustitución de objetos.

3. Proponga el nombre de la interfaz (abstracción) que debería interponerse entre
EstimadorPesoService y el cliente HTTP.
IEstimadorML.