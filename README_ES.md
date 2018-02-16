
# JAVA 8 - Cheat Sheet

## Expresion Lambda
```java
(int a) -> a * 2; // Calcula el docble de a
a -> a * 2; // o simplemente sin tipo
```
```java
(a, b) -> a + b; // Suma 2 parametros
```

Si lambda tiene mas de una expresion podemos usar `{ }` y `return`

```java
(x, y) -> {
	int sum = x + y;
	int avg = sum / 2;
	return avg;
}
```

Una expresion lambda no puede usarse sola en Java, necesita estar asociada a un interfaz funcional

```java
interface MyMath {
    int getDoubleOf(int a);
}
	
MyMath d = a -> a * 2; // asociado a un interfaz
d.getDoubleOf(4); // es 8
```

---

Todos los ejemplos con "list" usan:

```java
List<String> list = [Bohr, Darwin, Galilei, Tesla, Einstein, Newton]
```


## Colecciones

**sort** `sort(list, comparator)`

```java
list.sort((a, b) -> a.length() - b.length())
list.sort(Comparator.comparing(n -> n.length())); // igual
list.sort(Comparator.comparing(String::length)); // igual
//> [Bohr, Tesla, Darwin, Newton, Galilei, Einstein]
```

**removeIf**

```java
list.removeIf(w -> w.length() < 6);
//> [Darwin, Galilei, Einstein, Newton]
```

**merge**
`merge(key, value, remappingFunction)`

```java
Map<String, String> names = new HashMap<>();
names.put("Albert", "Ein?");
names.put("Marie", "Curie");
names.put("Max", "Plank");

// El valor "Albert" existe
// {Marie=Curie, Max=Plank, Albert=Einstein}
names.merge("Albert", "stein", (old, val) -> old.substring(0, 3) + val);

// El valor  "Newname" no existe
// {Marie=Curie, Newname=stein, Max=Plank, Albert=Einstein}
names.merge("Newname", "stein", (old, val) -> old.substring(0, 3) + val);
```


## Expresion de metodo `Class::staticMethod`

Permite referenciar metodos (y constructores) sin ejecutarlos

```java
// Con Lambda:
getPrimes(numbers, a -> StaticMethod.isPrime(a));

// Metodo Referenciado:
getPrimes(numbers, StaticMethod::isPrime);
```

| Metodo Referenciado | Forma de Lambda |
| ------------------- | --------------- |
| `StaticMethod::isPrime` | `n -> StaticMethod.isPrime(n)` |
| `String::toUpperCase`   | `(String w) -> w.toUpperCase()` |
| `String::compareTo`     | `(String s, String t) -> s.compareTo(t)` |
| `System.out::println`   | `x -> System.out.println(x)` |
| `Double::new`           | `n -> new Double(n)` |
| `String[]::new`         | `(int n) -> new String[n]` |


## Streams

Similares a las colecciones, pero:

 - No almacenan su propia informacion
 - La informacion viene de otra parte (colleciones, archivos, db, web, ...)
 - *immutable* (crean un nuevo stream)
 - *lazy* (solo computa lo que es necesario !)
 
```java
// Computara 3 "filter"
Stream<String> longNames = list
   .filter(n -> n.length() > 8)
   .limit(3);
```

**Crea un nuevo stream**

```java
Stream<Integer> stream = Stream.of(1, 2, 3, 5, 7, 11);
Stream<String> stream = Stream.of("Jazz", "Blues", "Rock");
Stream<String> stream = Stream.of(myArray); // or from an array
list.stream(); // or from a list

// Stream infinito [0; inf[
Stream<Integer> integers = Stream.iterate(0, n -> n + 1);
```

**Resultado de las colecciones**

```java
// Collect into an array (::new is the constructor reference)
// Colecciona en un array (::new es la referencia del contructor)
String[] myArray = stream.toArray(String[]::new);

// Collecciona en un List o Set
List<String> myList = stream.collect(Collectors.toList());
Set<String> mySet = stream.collect(Collectors.toSet());

// Collecciona en un String
String str = list.collect(Collectors.joining(", "));
```

**map** `map(mapper)`<br>
Aplica una funcion a cada elemento

```java
// Aplica "toLowerCase" a cada elemento
res = stream.map(w -> w.toLowerCase());
res = stream.map(String::toLowerCase);
//> bohr darwin galilei tesla einstein newton

res = Stream.of(1,2,3,4,5).map(x -> x + 1);
//> 2 3 4 5 6
```

**filter** `filter(predicado)`<br>
Retiene elementos que coinciden con el predicado

```java
// Filtra elementos que empiecen con "E"
res = stream.filter(n -> n.substring(0, 1).equals("E"));
//> Einstein

res = Stream.of(1,2,3,4,5).filter(x -> x < 3);
//> 1 2
```

**reduce**<br>
Reduce los elementos a un unico valor

```java
String reduced = stream
	.reduce("", (acc, el) -> acc + "|" + el);
//> |Bohr|Darwin|Galilei|Tesla|Einstein|Newton
```

**limit** `limit(maxSize)`
Los n primeros elementos

```java
res = stream.limit(3);
//> Bohr Darwin Galilei
```

**skip**
Descarta los primero n elementos

```java
res = strem.skip(2); // skip Bohr and Darwin
//> Galilei Tesla Einstein Newton
```

**distinct**
Borra los elementos repetidos

```java
res = Stream.of(1,0,0,1,0,1).distinct();
//> 1 0
```

**sorted**
Ordena elementos (debe ser *Comparable*)

```java
res = stream.sorted();
//> Bohr Darwin Einstein Galilei Newton Tesla 
```

**allMatch**

```java
// Comprueba si hay una "e" en cada elemento
boolean res = words.allMatch(n -> n.contains("e"));
```

anyMatch: Comprueba si hay una "e" en algun elemento<br>
noneMatch: Comprueba si no hay una "e" en ningun elemento

**parallel**
Devuelve un stream equivalente que es paralelo

**findAny**
mas rapido que findFirst en un stream paralelo

### Streams de tipo primitivo

Los wrappers (como Stream<Integer>) son ineficientes. Requieren de embalar y desembalar cada elemento demsiado. Mejor usar `IntStream`, `DoubleStream`, etc.

**Creacion**

```java
IntStream stream = IntStream.of(1, 2, 3, 5, 7);
stream = IntStream.of(myArray); // de un array
stream = IntStream.range(5, 80); // rango de 5 a 80

Random gen = new Random();
IntStream rand = gen(1, 9); // stream de aleatorios
```

Usa *mapToX* (mapToObj, mapToDouble, etc.) si la funcion produce un valor Object, double, etc.

### Resultados agrupados

**Collectors.groupingBy**

```java
// Agrupados por longitud
Map<Integer, List<String>> groups = stream
	.collect(Collectors.groupingBy(w -> w.length()));
//> 4=[Bohr], 5=[Tesla], 6=[Darwin, Newton], ...
```

**Collectors.toSet**

```java
// Igual que antes pero en un Set
... Collectors.groupingBy(
	w -> w.substring(0, 1), Collectors.toSet()) ...
```

**Collectors.counting**
Cuenta el numero de elementos en una coleccion

**Collectors.summing__**
`summingInt`, `summingLong`, `summingDouble` para sumar valores de un grupo

**Collectors.averaging__**
`averagingInt`, `averagingLong`, ... 

```java
// Longitud promedio de cada elemento de un grupo
Collectors.averagingInt(String::length)
```

*PS*: No olvides Optional (como `Map<T, Optional<T>>`) con algunos metodos de Colecciones (like `Collectors.maxBy`).


### Streams Paralelos

**Creacion**

```java
Stream<String> parStream = list.parallelStream();
Stream<String> parStream = Stream.of(myArray).parallel();
```

**unordered**
Pueden acelerar el `limit` o `distinct`

```java
stream.parallelStream().unordered().distinct();
```

*PS*: Trabaja con la libreria de streams. Pe. usa `filter(x -> x.length() < 9)` en vez de `forEach` con un`if`.
*PS*: Work with the streams library. Eg. use `filter(x -> x.length() < 9)` instead of a `forEach` with an `if`.


## Optional
En Java, es comun usar null para denota ausencia de resutlado.
Problemas cuando no se comprueban: `NullPointerException`.

```java
// Optional<String> contiene un string o nada
Optional<String> res = stream
   .filter(w -> w.length() > 10)
   .findFirst();

// longitud de res o "", si no trae nada
int length = res.orElse("").length();

// lanza el lambda si no trae nada
res.ifPresent(v -> results.add(v));
```

Devuelve un Optional

```java
Optional<Double> squareRoot(double x) {
   if (x >= 0) { return Optional.of(Math.sqrt(x)); }
   else { return Optional.empty(); }
}
```

---

**Note en la inferencia de la limitacion**

```java
interface Pair<A, B> {
    A first();
    B second();
}
```

Un stream de tipo `Stream<Pair<String, Long>>` :

 - `stream.sorted(Comparator.comparing(Pair::first)) // vale`
 - `stream.sorted(Comparator.comparing(Pair::first).thenComparing(Pair::second)) // no funciona`

Java cannot infer type for the `.comparing(Pair::first)` part y volver al Objeto, por lo que `Pair::first` no podria ser aplicado.

El tipo requerido para toda la expresion no puede ser propagada a traves de la llamada del metodo (`.thenComparing`) y  ser usada para inferir el tipo de la primera parte.

El tipo *debe* ser dado explicitamente
Type *must* be given explicitly.

```java
stream.sorted(
    Comparator.<Pair<String, Long>, String>comparing(Pair::first)
    .thenComparing(Pair::second)
) // ok
```

---

Esta cheat sheet esta basada en la leccion de Cay Horstmann
http://horstmann.com/heig-vd/spring2015/poo/
