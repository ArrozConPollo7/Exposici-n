# GUÍA DE ESTUDIO: VERIFICACIÓN FORMAL DE BFS MEDIANTE LÓGICA DE HOARE

## 1. Introducción y Conceptos Fundamentales

### ¿Qué es BFS (Breadth-First Search)?
BFS es un algoritmo de exploración de grafos que recorre los nodos por **niveles**. Partiendo de un nodo origen $s$, visita primero todos sus vecinos (distancia 1), luego los vecinos de estos (distancia 2), y así sucesivamente.

### ¿Qué es la Lógica de Hoare?
Es un sistema formal para razonar sobre la corrección de programas. Se basa en la **Terna de Hoare**:
$$\{P\} \, S \, \{Q\}$$
- **$P$ (Precondición):** Lo que debe ser cierto antes de ejecutar el programa.
- **$S$ (Programa):** El código o algoritmo.
- **$Q$ (Postcondición):** Lo que el programa garantiza al terminar.

---

## 2. La Terna de Hoare para BFS

### Precondición ($P$)
- El grafo $G = (V, E)$ es no dirigido.
- Los nodos origen $s$ y destino $t$ pertenecen a $V$.
- El conjunto de vértices $V$ no es vacío.

### Postcondición ($Q$)
- El algoritmo retorna la variable `encontrado`.
- Se garantiza que: `encontrado = true` $\iff$ existe un camino entre $s$ y $t$.

---

## 3. Invariantes de Bucle (El Corazón de la Prueba)

Un invariante es una propiedad que es cierta **antes**, **durante** y **después** de cada iteración del bucle.

### $I_1$: Alcanzabilidad (Seguridad)
$$\forall v \in \text{visitado}, \text{alcanzable}(s, v)$$
*Significado:* El algoritmo nunca marca un nodo como "visitado" a menos que realmente pueda llegar a él desde el origen.

### $I_2$: Completitud (Frontera de Exploración)
$$\text{alcanzable}(s, t) \implies (t \in \text{visitado} \lor \exists u \in \text{cola} : \text{camino}(u, t) \cap \text{visitado} = \emptyset)$$
*Significado:* Si existe un camino a $t$, o bien ya lo encontramos ($t \in \text{visitado}$), o bien hay un nodo en la cola que sirve de "puente" hacia $t$ a través de nodos no visitados. **Este invariante garantiza que no "perderemos" el camino a $t$.**

### $I_3$: Consistencia (Identidad de éxito)
$$\text{encontrado} \iff (t \in \text{visitado})$$
*Significado:* La bandera de éxito es verdadera si y solo si el nodo destino ha sido alcanzado y registrado.

---

## 4. El Paso Inductivo

Para demostrar que los invariantes se mantienen, analizamos qué sucede cuando procesamos un nodo `actual` y su vecino `v`.

### Caso A: $v \neq t$
1. **$I_1$:** Si `actual` es alcanzable (por HI) y hay un arco $(actual, v)$, entonces `v` es alcanzable. $I_1$ se mantiene.
2. **$I_2$:** Al encolar `v`, expandimos la frontera. Si $t$ era alcanzable a través de `actual`, ahora lo es a través de `v`. $I_2$ se mantiene.
3. **$I_3$:** `encontrado` sigue siendo `false` y $t$ no está en `visitado`. $\text{false} \iff \text{false}$. $I_3$ se mantiene.

### Caso B: $v = t$
1. **$I_1$:** $t$ es alcanzable vía `actual`. $I_1$ se mantiene.
2. **$I_3$:** `encontrado` pasa a `true` y $t$ entra en `visitado`. $\text{true} \iff \text{true}$. $I_3$ se mantiene.
3. **$I_2$:** Como `encontrado` es `true`, el bucle termina. $I_2$ se cumple vacuamente.

---

## 5. Terminación

No basta con que el algoritmo sea correcto, debe **terminar**.
- **Función de Cota:** $f = |V| - |\text{visitado}|$.
- En cada iteración donde descubrimos un nodo nuevo, $|\text{visitado}|$ aumenta, por lo que $f$ disminuye.
- Como $f$ no puede ser menor que 0 y el grafo es finito ($|V| < \infty$), el algoritmo terminará obligatoriamente.

---

## 6. Posibles Preguntas del Profesor

**P1: ¿Por qué es vital que BFS use una cola FIFO para la optimalidad?**
*Respuesta:* La cola FIFO garantiza que los nodos se procesen en el orden exacto de su distancia al origen. Todos los nodos a distancia $k$ se procesan antes que cualquier nodo a distancia $k+1$. Esto asegura que el primer camino encontrado a cualquier nodo sea siempre el más corto.

**P2: Si el grafo fuera infinito, ¿el algoritmo seguiría siendo correcto?**
*Respuesta:* El algoritmo sería "parcialmente correcto" (si termina, la respuesta es correcta). Sin embargo, si el nodo $t$ no es alcanzable, el algoritmo entraría en un bucle infinito buscando en una frontera que nunca se agota, por lo que no habría "corrección total".

**P3: Explique el invariante $I_2$. ¿Qué pasaría si no se cumpliera?**
*Respuesta:* $I_2$ asegura que no "cerremos" todas las vías hacia $t$ sin haberlo visitado. Si no se cumpliera, el algoritmo podría vaciar la cola pensando que exploró todo, cuando en realidad dejó una rama sin visitar que llevaba al destino. $I_2$ garantiza que la "antorcha" de la búsqueda siempre pase a un nodo en la cola.

**P4: ¿En qué momento exacto se establece la veracidad de la postcondición $Q$?**
*Respuesta:* Al salir del bucle. Si salimos porque `encontrado = true`, por $I_3$ sabemos que $t \in \text{visitado}$ y por $I_1$ que es alcanzable. Si salimos porque `cola = \emptyset`, por el contrapositivo de $I_2$ sabemos que $t$ no es alcanzable. En ambos casos, el valor de `encontrado` coincide con la existencia real del camino.

**P5: ¿Cuál es la diferencia entre Verificación Formal y Testing tradicional?**
*Respuesta:* El testing prueba el código con un conjunto finito de ejemplos (casos de prueba). La verificación formal usa lógica matemática para demostrar que el código es correcto para **cualquier** entrada posible que cumpla la precondición, ofreciendo una garantía absoluta del 100%.
