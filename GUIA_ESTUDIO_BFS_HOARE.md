# GUÍA DE ESTUDIO DE ALTA PRECISIÓN: VERIFICACIÓN FORMAL DE BFS
**Curso:** Teoría de la Computación  
**Universidad:** EAFIT, 2026  
**Autores:** Salomé Bedoya Echavarría, Juan David Giraldo Regino, Claudia Yesenia Castaño Quintero

---

## 1. Contexto General y Fundamientos Teóricos

### La Necesidad de la Verificación Formal
En el desarrollo de software convencional, la validez de un algoritmo se suele evaluar mediante pruebas dinámicas (testing). Sin embargo, el testing solo demuestra la presencia de errores, no su ausencia. En sistemas críticos, se requiere **verificación formal**: un método matemático estático para demostrar que un algoritmo es absolutamente correcto para cualquier conjunto de datos de entrada válido.

### La Lógica de Hoare y la Terna Canónica
La verificación formal de programas imperativos utiliza la Lógica de Hoare, la cual se estructura en torno a tripletas o ternas con la forma:
$$\{P\} \, S \, \{Q\}$$

*   **$P$ (Precondición):** Una aserción lógica sobre el estado inicial de las variables que debe ser verdadera antes de ejecutar el programa $S$.
*   **$S$ (Programa/Algoritmo):** La secuencia de instrucciones o código estructurado que transforma el estado de los datos.
*   **$Q$ (Postcondición):** Una aserción lógica sobre el estado final de las variables que se garantiza verdadera tras la terminación de $S$, siempre que $P$ haya sido inicialmente cierta.

### Corrección Parcial vs. Corrección Total
*   **Corrección Parcial:** Si el algoritmo arranca en un estado que satisface $P$ y logra terminar, se garantiza que el estado final cumplirá obligatoriamente con $Q$.
*   **Corrección Total:** Garantiza la corrección parcial y además demuestra matemáticamente que el algoritmo siempre terminará (es decir, que no entra en bucles infinitos).

---

## 2. Definición del Problema Específico

El objetivo de esta verificación es evaluar formalmente el algoritmo de Búsqueda en Anchura (BFS) diseñado para responder a un problema fundamental en la teoría de grafos: la alcanzabilidad binaria, determinando si existe un camino en $G$ desde un vértice origen $s$ hasta un destino $t$.

### Especificación Formal de la Terna de Hoare para BFS

**Precondición $P$:**
$$P \equiv G = (V, E) \text{ es un grafo no dirigido} \land V \neq \emptyset \land s, t \in V$$

**Postcondición $Q$:**
$$Q \equiv \text{encontrado} = \text{true} \iff \text{camino}(s, t) \text{ en } G$$

### El Algoritmo Estudiado (Sintaxis Formalizada)

```pseint
1:  // Precondición P
2:  visitado ← ∅
3:  cola ← Cola_Vacia()
4:  encontrado ← false
5:  visitado.agregar(s)
6:  cola.encolar(s)
7:  // Establecimiento de Invariantes I1, I2, I3
8:  while (cola ≠ ∅) y (not encontrado) do
9:      actual ← cola.desencolar()
10:     for cada vecino de actual do
11:         if vecino ∉ visitado then
12:             visitado.agregar(vecino)
13:             if vecino = t then
14:                 encontrado ← true
15:             else
16:                 cola.encolar(vecino)
17:             end if
18:         end if
19:     end for
20: end while
21: // Postcondición Q
```

> **Nota de Diseño Crítica:** La detección del vértice meta $t$ ocurre de manera radial al momento de encolar, no al desencolar. Esto garantiza que `encontrado` sea `true` exactamente cuando $t$ ingresa al conjunto `visitado`, evitando un estado transitorio inconsistente.

---

## 3. Demostración Matemática: Invariantes de Bucle

Para demostrar la corrección de un bucle se requiere el uso de **Invariantes de Bucle**: predicados lógicos que se mantienen verdaderos antes, durante y después de cada iteración del ciclo.

### Definición de los Tres Invariantes Colectivos

1.  **Invariante de Alcanzabilidad ($I_1$):**
    $$\forall v \in \text{visitado}, \text{alcanzable}(s, v)$$
    Todo nodo marcado como "visitado" posee un camino de acceso desde el origen $s$.

2.  **Invariante de Completitud de Frontera ($I_2$):**
    $$\text{alcanzable}(s, t) \implies (t \in \text{visitado} \lor \exists u \in \text{cola} : \text{camino}(u, t) \cap \text{visitado} = \emptyset)$$
    Si $t$ es alcanzable, o bien ya lo encontramos, o hay un nodo activo en la cola que sirve de "puente" hacia $t$ a través de vértices no visitados.

3.  **Invariante de Consistencia Simétrica ($I_3$):**
    $$\text{encontrado} = \text{true} \iff t \in \text{visitado}$$
    La bandera de éxito refleja exactamente la inclusión de la meta en el registro histórico.

### Paso A: Demostración del Caso Base (Inicialización)
*   **Para $I_1$:** Al inicio, $\text{visitado} = \{s\}$. Como un vértice está a distancia 0 de sí mismo, el camino trivial $(s)$ demuestra alcanzabilidad.
*   **Para $I_2$:** Si existe un camino de $s$ a $t$, el elemento inicial es $s$, el cual está en la cola, y el resto del camino no ha sido visitado. Tomando $u = s \in \text{cola}$, se cumple.
*   **Para $I_3$:** $\text{encontrado} = \text{false}$ y $t \notin \text{visitado}$ (asumiendo $s \neq t$). La equivalencia $\text{false} \iff \text{false}$ es válida.

### Paso B: Demostración del Paso Inductivo (Preservación)

**Subcaso 1: El vecino evaluado no es el nodo meta ($vecino \neq t$)**
1.  **Preservación de $I_1$:** Si `vecino` $\notin$ `visitado`, se añade. Como `actual` es alcanzable y existe $(actual, vecino)$, el nuevo vértice también lo es.
2.  **Preservación de $I_2$:** Al desencolar `actual`, la continuidad del camino se transfiere a sus vecinos no visitados ingresados a la cola. La frontera se expande.
3.  **Preservación de $I_3$:** `encontrado` sigue siendo `false` y $t \notin \text{visitado}$. $\text{false} \iff \text{false}$ permanece.

**Subcaso 2: El vecino evaluado es el nodo meta ($vecino = t$)**
1.  **Preservación de $I_1$:** $t$ es vecino de `actual` (alcanzable), por tanto $t$ es alcanzable.
2.  **Preservación de $I_2$:** Al activarse `encontrado`, el bucle terminará. La condición se vuelve vacuamente cierta.
3.  **Preservación de $I_3$:** Se añade $t$ a `visitado` y se cambia `encontrado` simultáneamente. $\text{true} \iff \text{true}$.

---

## 4. Demostración de Terminación y Corrección Total

Utilizamos una función de cota (o variante de bucle) $f: \mathcal{P}(V) \to \mathbb{N}$:
$$f(\text{visitado}) = |V| - |\text{visitado}|$$

1.  **Acotación Inferior:** Como $|\text{visitado}| \le |V|$, entonces $f(\text{visitado}) \ge 0$.
2.  **Decremento Estricto:** En cada iteración donde no termina por éxito, al menos un vértice de $V \setminus \text{visitado}$ es transferido a `visitado`.
3.  **Conclusión:** $|visitado_{k+1}| > |visitado_k| \implies f(visitado_{k+1}) < f(visitado_k)$. Al decrementar estrictamente sobre un conjunto finito, el algoritmo termina inevitablemente.

---

## 5. El Ejemplo Canónico: La Bifurcación de Control

**Estructura del Grafo:**
*   **Vértices:** $\{s(A), B, C, D, t(E)\}$
*   **Aristas:** $\{(s, B), (s, C), (B, D), (C, t), (D, t)\}$

**Traza Paso a Paso:**
1.  **Paso 0:** $\text{visitado}=\{s(A)\}, \text{cola}=[s(A)]$. Invariantes OK.
2.  **Paso 1 (Procesar $s$):** $\text{visitado}=\{s, B, C\}, \text{cola}=[B, C]$. Se encolan vecinos.
3.  **Paso 2 (Procesar $B$):** Se encola $D$. $\text{cola}=[C, D]$. El nodo $C$ (ruta óptima) queda al frente.
4.  **Paso 3 (Procesar $C$):** Se detecta $t(E)$. $\text{encontrado} \gets \text{true}$. El bucle se rompe. La ruta larga por $D$ queda descartada, demostrando optimalidad.

---

## 6. Banco de Preguntas de Examen

### Bloque A: Técnicas y Código
**P1: ¿Por qué la postcondición $Q$ es un bicondicional ($\iff$)?**
*R: Exige tanto Corrección (si dice true, el camino existe) como Completitud (si existe, debe encontrarlo).*

**P2: ¿Qué ocurre si 'encontrado' se activa al desencolar?**
*R: Rompe $I_3$. Habría estados donde $t \in \text{visitado}$ pero `encontrado` es `false`, destruyendo la validez de la prueba de bucle.*

**P3: ¿Es correcto eliminar 'not encontrado' de la guarda?**
*R: Sería parcialmente correcto pero ineficiente. El algoritmo exploraría todo el grafo innecesariamente tras hallar la meta.*

### Bloque B: Lógica e Invariantes
**P4: ¿Relación de $I_2$ con la optimalidad?**
*R: Formaliza la "frontera activa". Como la cola es FIFO, asegura que procesamos por niveles de distancia. $I_2$ garantiza que siempre avanzamos nivel por nivel hacia $t$.*

**P5: ¿Por qué evaluar el subcaso $vecino = t$ por separado?**
*R: Porque ocurre una mutación sincrónica del estado (éxito) que detiene el bucle, a diferencia de la expansión normal.*

### Bloque C: Abstractas y Frontera
**P6: ¿Qué ocurre en un grafo infinito?**
*R: Si $t$ es alcanzable, termina (distancia finita). Si no es alcanzable, entra en bucle infinito y falla la corrección total.*

**P7: Hoare vs. Inducción Clásica.**
*R: La inducción clásica prueba propiedades estructurales abstractas. Hoare certifica el estado real de los registros del software paso a paso.*
