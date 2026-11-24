# Las fórmulas, término a término — Ranking Sort (CS4052)

Versión escrita del video [Las fórmulas, término a término](media/videos/formulas.mp4) ([solo audio](media/audio/formulas.m4a)): de dónde sale cada fórmula del análisis, qué significa cada término y cómo se llega a ella. Casi todo se deriva de una sola expresión, el cómputo de un proceso $T_{\mathrm{comp}}$.

## 1. Los símbolos

- $n$ — cantidad de claves a ordenar
- $p = q^2$ — cantidad de procesos, en una grilla de q × q

Primero, los símbolos. n es la cantidad de claves a ordenar. p es la cantidad de procesos, y debe ser un cuadrado perfecto, p = q², para formar una grilla de q × q.

- $x = \frac{n}{\sqrt{p}}$ — tamaño del bloque de cada proceso
$$\frac{n}{p} \qquad\times\qquad \sqrt{p} \qquad=\qquad \frac{n}{\sqrt{p}}$$

- $\frac{n}{p}$ : Scatter: lo que recibe cada uno
- $\sqrt{p}$ : Gossip: trozos de la columna
- $\frac{n}{\sqrt{p}}$ : bloque de columna

Equis es el tamaño del bloque con el que trabaja cada proceso: n/√p. ¿Por qué √p y no p? Porque el Scatter le da n/p claves a cada uno, pero el Gossip junta los √p trozos de una columna. √p por n/p es n/√p.

- $t_c$ — una comparación: la unidad de tiempo (Sort y Local ranking)

La unidad de tiempo es t_c: lo que cuesta una comparación. Solo dos fases comparan: el Sort y el Local ranking. Todo lo demás es comunicación.

- $\alpha + \beta\, m$ — costo de un mensaje de m pares
- $\alpha,\ \beta,\ \gamma$ — latencia  ·  costo por par  ·  costo de una suma

La comunicación usa el modelo α β: un mensaje de m pares cuesta α más β por m. α es la latencia, lo que cuesta arrancar un mensaje. β es lo que cuesta cada par transmitido. Y γ es lo que cuesta una suma en el Reduce.

## 2. La base: el cómputo de un proceso

$$T_{\mathrm{comp}}\;=\; x\lceil\log_2 x\rceil \;+\; x\,(\lceil\log_2 x\rceil+1)$$

- $x\lceil\log_2 x\rceil$ : Sort (cota de mergesort)
- $x\,(\lceil\log_2 x\rceil+1)$ : Local ranking: x búsquedas binarias

La fórmula base es el tiempo de cómputo de un proceso, T_comp. Tiene dos sumandos, uno por cada fase que compara.

- log₂ x niveles de mezcla (redondeado hacia arriba) × a lo más x comparaciones por nivel

El primero es el Sort. Ordenar x elementos cuesta a lo más x por ⌈log x⌉ comparaciones. Es la cota de mergesort: hay log de x niveles de mezcla, y en cada nivel se hacen a lo más x comparaciones. El techo aparece porque la cantidad de niveles es un entero.

- cada búsqueda: log₂ x pasos partiendo a la mitad (redondeado) + 1 comparación final

El segundo es el Local ranking: x búsquedas binarias, una por cada elemento del bloque de la fila. Cada búsqueda parte el intervalo a la mitad en cada paso, así que hace ⌈log x⌉ comparaciones, más una final. Por eso es ⌈log x⌉ más uno.

$$T_{\mathrm{comp}}\;=\; x \;\cdot\; (2\lceil\log_2 x\rceil+1)$$

- $x$ : elementos del bloque
- $(2\lceil\log_2 x\rceil+1)$ : costo por elemento: ordenar + buscar

Sumando y sacando x como factor común: x por, 2⌈log x⌉, más uno. Se lee así: x elementos, y cada uno cuesta dos log más uno. Un log por ordenarlo, y un log más uno por buscarlo.

$$= \Theta\left(\frac{n}{\sqrt{p}}\,\log\frac{n}{\sqrt{p}}\right)$$

> **Ejemplo:** n = 18, p = 9:   x = 6,   6 · (2·3 + 1) = 42 comparaciones en cada proceso

En notación asintótica es theta de n/√p por el log de lo mismo. En el ejemplo del enunciado, con dieciocho claves y nueve procesos, x es seis y ⌈log 6⌉ es tres: seis por siete, cuarenta y dos comparaciones. Y como no depende de los valores, todos los procesos hacen exactamente lo mismo.

## 3. El tiempo secuencial

$$T_s = T_{\mathrm{comp}}(n,\,1)$$

La referencia para medir el speedup es el tiempo secuencial, Tₛ. No es otra fórmula: es la misma de T_comp, con p igual a uno.

$$T_s =  n \,(2\lceil\log_2 n\rceil+1) \;=\; \Theta(n\,\log n)$$

- $n$ : x = n: el bloque es todo el arreglo
- $\Theta(n\,\log n)$ : ordenar + buscar

Con un proceso, la grilla es de uno por uno, el bloque es todo el arreglo, x igual a n, y las colectivas no hacen nada. Queda n por, 2⌈log n⌉, más uno: theta de n log n.

- $T_s^{RS} = n^2$ — Ranking Sort clásico: todos contra todos

Ojo: con un proceso, el algoritmo no compara todos contra todos. Ordena el arreglo y busca cada elemento en él. Por eso cuesta n log n, como el mejor ordenamiento por comparaciones, y no n², que es el Ranking Sort clásico.

> **Ejemplo:** n = 18:   Tₛ = 18 · (2·5 + 1) = 198      frente a      n² = 324

Esto importa. Medir contra n log n es la comparación honesta. Si se midiera contra n², el speedup saldría inflado, porque se estaría comparando con un algoritmo peor. En el ejemplo: ciento noventa y ocho comparaciones, contra trescientas veinticuatro.

## 4. Trabajo y redundancia

$$W =  p \;\cdot\; T_{\mathrm{comp}}$$

- $p$ : procesos
- $T_{\mathrm{comp}}$ : lo que hace cada uno

El trabajo, W, es la suma de las operaciones de todos los procesos. Como todos hacen T_comp, es simplemente p por T_comp.

$$W =  n\sqrt{p} \,(2\lceil\log_2\frac{n}{\sqrt{p}}\rceil+1)$$

- $n\sqrt{p}$ : = p · n/√p

p por n/√p es n·√p. Entonces el trabajo es n √p, por el factor del logaritmo. Fíjate: crece con p. Agregar procesos aumenta el trabajo total.

$$\frac{W}{T_s} =  \sqrt{p} \;\cdot\; \lambda$$

- $\sqrt{p}$ : redundancia
- $\lambda$ : corrección por logaritmos

Para medir cuánto sobra, se divide entre el secuencial. Las n se cancelan, y queda √p por λ.

$$\lambda = \frac{2\lceil\log_2\frac{n}{\sqrt{p}}\rceil+1}{2\lceil\log_2 n\rceil+1}$$

- n/√p < n   ⇒   numerador menor   ⇒   λ ≤ 1
- p ≤ n   ⇒   n/√p ≥ √n   ⇒   el logaritmo baja a la mitad como mucho   ⇒   λ ≳ 1/2

λ es el cociente entre el costo por elemento en paralelo y en secuencial. Como n/√p es menor que n, el numerador es menor: λ es a lo más uno. Y como p no supera a n, el bloque tiene al menos √n elementos, cuyo logaritmo es la mitad del de n: λ es al menos un medio, aproximadamente.

> **Ejemplo:** n = 32·10⁶, p = 16:   λ = (2·23 + 1)/(2·25 + 1) = 47/51 ≈ 0,92   ⇒   W/Tₛ = 4 · 0,92 ≈ 3,7

Con n mucho mayor que p, λ tiende a uno, y la redundancia es √p. Con treinta y dos millones de claves y dieciséis procesos, λ es cuarenta y siete sobre cincuenta y uno, cero noventa y dos: se hace tres coma siete veces el trabajo necesario.

- cada bloque Bⱼ se ordena √p veces (una por proceso de la columna)
- cada elemento se busca en √p bloques (los de su fila)

¿De dónde viene ese √p? De dos lugares: cada bloque se ordena en los √p procesos de su columna, y cada elemento se busca en los √p bloques de su fila. Es inherente al esquema, no un error de implementación. Y es lo que explica todas las fórmulas que siguen.

## 5. Camino crítico y paralelismo

$$p \leq n \;\Rightarrow\; p = n:\quad x = \frac{n}{\sqrt{n}} = \sqrt{n}$$

El camino crítico es el tiempo con el máximo de procesos que admite el diseño. Como cada proceso debe recibir al menos una clave, p no puede pasar de n. Con p igual a n, el bloque tiene √n elementos.

$$T_\infty = T_{\mathrm{comp}}(n,n) =  \sqrt{n} \;\; (2\lceil\log_2\sqrt{n}\rceil+1) \;=\;\Theta(\sqrt{n}\,\log n)$$

- $\sqrt{n}$ : bloque
- $(2\lceil\log_2\sqrt{n}\rceil+1)$ : costo por elemento

Reemplazando en T_comp: T∞ es √n por, 2⌈log √n⌉, más uno. Theta de √n log n. Es lo mínimo que puede tardar el cómputo, aunque sobren procesos.

$$\frac{W(n,n)}{T_\infty} = \frac{n\sqrt{n}\,(2\lceil\log_2\sqrt{n}\rceil+1)}{\sqrt{n}\,(2\lceil\log_2\sqrt{n}\rceil+1)} = n$$

- el diseño aprovecha hasta n procesos (p ≤ n)

El paralelismo es el trabajo sobre el camino crítico. Con p igual a n, el trabajo es n por √n por el mismo factor, y al dividir queda exactamente n. Significa que el diseño puede aprovechar hasta n procesos: coincide con la restricción p ≤ n.

## 6. El tiempo de comunicación

- cada colectiva: latencia (α) + volumen (β); sobre m procesos, log₂ m rondas (árbol)

Ahora la comunicación. Son cuatro colectivas, y cada una tiene un término de latencia, con α, y uno de volumen, con β. Las colectivas usan árboles: sobre m procesos hacen log m rondas.

$$\mathrm{Scatter}:\; \alpha\lceil\log_2 p\rceil \;+\; \beta\left(n-\frac{n}{p}\right)$$

- $\alpha\lceil\log_2 p\rceil$ : latencia: log p rondas
- $\beta\left(n-\frac{n}{p}\right)$ : todo menos lo suyo, por un puerto

El Scatter: α·log p, porque el árbol tiene log p rondas. Y β por n menos n/p, porque la raíz tiene que sacar todas las claves menos las suyas, y tiene un solo puerto. El árbol reparte el reenvío, pero no reduce lo que sale de la raíz.

$$\mathrm{Gossip}:\;\alpha\lceil\log_2\sqrt{p}\rceil\;+\; \beta\left(\frac{n}{\sqrt{p}}-\frac{n}{p}\right)$$

- $\beta\left(\frac{n}{\sqrt{p}}-\frac{n}{p}\right)$ : bloque de columna menos lo propio

El Gossip es un All-gather en la columna, de √p procesos: cada uno recibe el bloque de la columna menos su parte, n/√p menos n/p.

$$\mathrm{Bcast}:\; \lceil\log_2\sqrt{p}\rceil \;\; \left(\alpha+\beta\frac{n}{\sqrt{p}}\right)$$

- $\lceil\log_2\sqrt{p}\rceil$ : rondas en la fila
- $\left(\alpha+\beta\frac{n}{\sqrt{p}}\right)$ : bloque completo en cada ronda

$$\mathrm{Reduce}:\;\lceil\log_2\sqrt{p}\rceil\left(\alpha+(\beta+\gamma)\frac{n}{\sqrt{p}}\right)$$

El Broadcast y el Reduce trabajan en la fila. En cada una de las log √p rondas se retransmite el bloque entero, por eso el término es log √p, por α más β·n sobre √p. El Reduce además suma, y por eso aparece γ.

$$\boxed{T_{\mathrm{comm}} = \Theta(\alpha\,\log p + \beta\, n)}$$

**el término dominante es el Scatter: βn, que no baja con p**

Sumando todo, la comunicación es del orden de α log p más β·n. El término dominante es el Scatter, β por n, que no baja con p. Es la parte secuencial del algoritmo, porque la entrada empieza en un solo proceso.

## 7. El tiempo paralelo

$$T_p =  T_{\mathrm{comp}} \;+\; T_{\mathrm{comm}}$$

- $T_{\mathrm{comp}}$ : cómputo
- $T_{\mathrm{comm}}$ : comunicación

El tiempo paralelo es el del proceso más lento, desde el Scatter hasta el Reduce. Como todos hacen las mismas colectivas y el mismo cómputo, es simplemente T_comp más T_comm.

$$T_p =  \frac{W}{p} \;+\;T_{\mathrm{comm}}$$

- $\frac{W}{p}$ : reparto perfecto del trabajo

Y como el trabajo es p por T_comp, T_comp es exactamente W/p. Entonces el tiempo paralelo es W/p, más la comunicación.

- balance perfecto: todos los procesos hacen exactamente lo mismo
- todo lo que excede a W/p es comunicación

Esa forma dice dos cosas. Primera: el cómputo está perfectamente repartido, no hay desbalance. Segunda: todo lo que exceda a W/p es comunicación.

$$T_p \;\sim\; \frac{n\,\log n}{\sqrt{p}} \;+\; \alpha\,\log p \;+\; \beta\, n$$

- $\frac{n\,\log n}{\sqrt{p}}$ : baja como 1/√p
- $\alpha\,\log p$ : sube
- $\beta\, n$ : fijo

En orden asintótico: n log n/√p, que baja con p; más α log p, que sube; más β·n, que se queda fijo. Toda la escalabilidad se lee de esta línea.

- Gather + colocación = T_post: se reporta aparte (no baja con p)

El Gather final y la colocación de la salida van aparte, como post procesamiento, porque los hace solo una parte de los procesos y no bajan con p.

## 8. El speedup

$$S(p) = \frac{T_s}{T_p}$$

El speedup es cuántas veces más rápido es el paralelo que el secuencial: Tₛ sobre Tₚ.

$$S_{\mathrm{comp}} = \frac{T_s}{T_{\mathrm{comp}}} = \frac{T_s}{W/p} = p\cdot\frac{T_s}{W}$$

Veamos primero solo el cómputo. S_comp es Tₛ sobre T_comp. Y como T_comp es W/p, es p por Tₛ sobre W.

$$S_{\mathrm{comp}} =  p \;\cdot\; \frac{1}{\sqrt{p}\,\lambda} \;=\; \frac{\sqrt{p}}{\lambda}$$

- $p$ : ideal
- $\frac{1}{\sqrt{p}\,\lambda}$ : Tₛ/W: lo que se pierde por redundancia
- $\frac{\sqrt{p}}{\lambda}$ : speedup de cómputo

Pero Tₛ sobre W es el inverso de la redundancia: 1/(√p·λ). Entonces S_comp es p sobre √p·λ, es decir, √p/λ.

- el ideal p se divide entre la redundancia √p·λ   ⇒   crece como √p, no como p

Esta es la derivación clave. El speedup ideal sería p, pero la redundancia se come un factor √p. Por eso el speedup crece como √p, y no como p.

$$\boxed{\sqrt{p}\;\leq\;S_{\mathrm{comp}} = \frac{\sqrt{p}}{\lambda}\;\lesssim\;2\sqrt{p}}$$

**porque  1/2 ≲ λ ≤ 1**

Como λ está entre un medio y uno, S_comp está entre √p y dos √p. El uno sobre λ ayuda un poco: los bloques más chicos tienen logaritmos más chicos.

> **Ejemplo:** n = 32·10⁶, p = 16:   S_comp = 4 / 0,92 ≈ 4,34      medido en comparaciones: 4,34

Con treinta y dos millones de claves y dieciséis procesos: cuatro sobre cero noventa y dos, cuatro coma tres cuatro. El contador de comparaciones midió exactamente cuatro coma tres cuatro.

## 9. Speedup real y su techo

$$S(p) = \frac{T_s}{T_{\mathrm{comp}} + T_{\mathrm{comm}}}$$

Con la comunicación, el speedup real es Tₛ sobre T_comp más T_comm. Se puede escribir como S_comp, por T_comp sobre Tₚ.

$$S(p) =  S_{\mathrm{comp}} \;\cdot\; \frac{T_{\mathrm{comp}}}{T_p}$$

- $S_{\mathrm{comp}}$ : límite por redundancia
- $\frac{T_{\mathrm{comp}}}{T_p}$ : fracción calculando (< 1)

Ese segundo factor es la fracción del tiempo que se dedica a calcular. Es menor que uno, así que el speedup real siempre es menor que el de cómputo.

$$S(p)\;\leq\;\frac{T_s}{\beta\,(n-n/p)}\;\approx\;\frac{n\cdot 2\log_2 n}{\beta\, n}\;=\;\frac{2\log_2 n}{\beta}$$

Y hay un techo. El tiempo paralelo nunca es menor que el Scatter, β por n menos n/p. Entonces el speedup no puede pasar de Tₛ sobre eso. Las n se cancelan, y queda aproximadamente 2 log n sobre β: un número que no depende de p.

> **Ejemplo:** n = 32·10⁶:   techo ≈ 67      ·      con p = 16:  S_comp teórico 4,34,  S medido 1,13

Con los valores medidos, ese techo es unas sesenta y siete veces, muy lejos de lo alcanzado. Con dieciséis procesos, el speedup real fue uno coma uno tres, porque los procesos compiten por la memoria del equipo. La fórmula supone que una comparación cuesta lo mismo con cualquier p.

## 10. La eficiencia

$$E(p) = \frac{S(p)}{p} = \frac{T_s}{p\,T_p}$$

La eficiencia es el speedup dividido entre p: qué fracción del ideal se aprovecha. También es Tₛ, sobre p por Tₚ.

$$E_{\mathrm{comp}} = \frac{\sqrt{p}/\lambda}{p} = \frac{1}{\sqrt{p}\,\lambda} = \Theta\left(\frac{1}{\sqrt{p}}\right)$$

Solo con cómputo: √p/λ, dividido entre p, da 1/(√p·λ). Theta de 1/√p: cae siempre al agregar procesos.

$$E(p) =  \frac{T_s}{W} \;\cdot\; \frac{T_{\mathrm{comp}}}{T_p}$$

- $\frac{T_s}{W}$ : redundancia = 1/(√p·λ)
- $\frac{T_{\mathrm{comp}}}{T_p}$ : fracción calculando

La forma más útil de leerla es como producto de dos factores. El primero, Tₛ sobre W, es la redundancia: depende solo del algoritmo, no de la red ni de la máquina.

- aunque T_comm = 0:   E ≤ 1/(√p·λ)

El segundo, T_comp sobre Tₚ, es la fracción del tiempo calculando, y mide el peso de la comunicación. Aunque la red fuera infinitamente rápida, el primer factor seguiría ahí. Por eso la eficiencia no puede mantenerse.

> **Ejemplo:** n = 32·10⁶:   p = 4 → E_comp ≈ 0,52 (medida 0,45)      ·      p = 16 → E_comp ≈ 0,27

Con cuatro procesos: uno sobre dos por cero noventa y seis, cerca de cero coma cinco dos. Se midió cero coma cuatro cinco, porque la comunicación se lleva el resto. Con dieciséis, la teoría da cero coma dos siete.

## 11. El costo

$$C = p\,T_p =  W \;+\; p\,T_{\mathrm{comm}}$$

- $W$ : trabajo
- $p\,T_{\mathrm{comm}}$ : procesos ocupados comunicando

El costo es p por Tₚ: el tiempo que los p procesos quedan ocupados. Como Tₚ es W/p más la comunicación, el costo es W, más p por T_comm.

$$C = \Theta\left(n\sqrt{p}\,\log n + \alpha\, p\,\log p + \beta\, p\, n\right)$$

- óptimo si C = Θ(n log n):   sobra un factor √p   ✗

Un algoritmo es óptimo en costo si su costo es del orden del mejor secuencial, n log n. Aquí el primer término ya es n·√p·log n: sobra un factor √p. No es óptimo en costo.

$$E = \frac{T_s}{C}$$

Y se conecta con la eficiencia: la eficiencia es Tₛ sobre el costo. Si el costo crece más rápido que Tₛ, la eficiencia tiene que caer. Es la misma conclusión, vista desde otro lado.

- frente al Ranking Sort clásico:   n·√p·log n  ≪  n²   ✓

Frente al Ranking Sort clásico de n², en cambio, el trabajo sí es menor, porque n·√p·log n es mucho menos que n².

## 12. ¿Cuántos procesos convienen?

$$f(p) =  n\,(2\log_2 n+\beta)\,p^{-1/2} \;+\; \frac{5}{2}\,\alpha\log_2 p$$

- $n\,(2\log_2 n+\beta)\,p^{-1/2}$ : baja: cómputo + volumen
- $\frac{5}{2}\,\alpha\log_2 p$ : sube: latencia

¿Cuántos procesos convienen? Se toman los términos del tiempo paralelo que dependen de p. Los que bajan son el cómputo y parte del volumen, que van como 1/√p. El que sube es la latencia, unos (5/2)·α log p.

$$f\,'(p) = -\frac{n\,(2\log_2 n+\beta)}{2}\,p^{-3/2} + \frac{5\,\alpha}{2\,p\,\ln 2} = 0$$

Se deriva respecto de p y se iguala a cero. La derivada de p^(−1/2) es menos un medio de p^(−3/2), y la de log p es uno sobre p por ln 2.

$$\boxed{\sqrt{p^*}\;\approx\;\frac{n\,(2\ln n+\beta\ln 2)}{5\,\alpha}}$$

**usando  log₂ n · ln 2 = ln n**

Despejando: √p* es aproximadamente n por, dos ln n más β ln 2, sobre 5α.

> **Ejemplo:** α = 10³ t_c,  n = 32 768   ⇒   √p* ≈ 136,   p* ≈ 1,9·10⁴ procesos

Con los valores del curso, α igual a mil comparaciones y n treinta y dos mil setecientos sesenta y ocho, sale √p* cerca de ciento treinta y seis: unos diecinueve mil procesos. Es solo indicativo, pero muestra que en un solo equipo el tiempo sigue bajando, y que lo que frena no es la latencia sino el piso del Scatter.

## 13. Escalabilidad fuerte

$$T_p = \frac{n\,\log n}{\sqrt{p}} + \alpha\,\log p + \beta\, n$$

Escalabilidad fuerte: n fijo, p creciendo. Se mira cada término del tiempo paralelo.

- p × 4  ⇒  cómputo ÷ 2      ·      Scatter fijo en βn      ·      latencia ↑ log p

El cómputo baja como 1/√p: cuadruplicar los procesos solo lo reduce a la mitad, porque se duplica el lado de la grilla. La comunicación no baja: el Scatter queda en β·n, y la latencia sube con log p.

*(gráfico en el video: speedup y eficiencia teóricos frente a lo medido)*

Resultado: el speedup crece como √p, con el techo del Scatter, y la eficiencia cae como 1/√p. Los puntos naranjas son lo medido: siguen la teoría hasta cuatro procesos y caen después por la memoria del equipo. El tiempo baja, pero cada proceso nuevo aporta menos.

## 14. Escalabilidad débil

Escalabilidad débil: n y p crecen juntos. Hay dos formas naturales de hacerlo.

$$\frac{n}{p} = m\;\Rightarrow\; \frac{n}{\sqrt{p}} = m\sqrt{p} \;\Rightarrow\;T_{\mathrm{comp}} = \Theta(\sqrt{p}\,\log p)$$

- $\frac{n}{\sqrt{p}} = m\sqrt{p}$ : el bloque crece

- no escala   ✗

Primera: fijar las claves iniciales por proceso, n/p igual a m. Pero el bloque que maneja cada proceso es n/√p, que es m·√p: crece. El cómputo por proceso es theta de √p log p, y el Scatter crece como p. No escala.

$$\frac{n}{\sqrt{p}} = c\;\Rightarrow\; T_{\mathrm{comp}} = c\,(2\lceil\log_2 c\rceil+1)$$

- $T_{\mathrm{comp}} = c\,(2\lceil\log_2 c\rceil+1)$ : constante

- el mejor régimen: solo crece la comunicación   ✓

Segunda: fijar el bloque, n/√p igual a c. Entonces T_comp es c por, dos log de c más uno: constante. Solo crece la comunicación. Es el mejor régimen.

$$\frac{n}{p} = \frac{c}{\sqrt{p}} \geq 1 \;\Rightarrow\; p \leq c^2$$

Pero tiene un límite: cada proceso recibe n/p, que es c/√p claves. Eso debe ser al menos uno, así que solo funciona hasta p igual a c al cuadrado.

## 15. Isoeficiencia

$$T_0 =  p\,T_p \;-\; T_s$$

- $p\,T_p$ : tiempo total ocupado
- $T_s$ : trabajo útil

La isoeficiencia pregunta: ¿a qué ritmo debe crecer n para mantener fija la eficiencia al aumentar p? Se define el overhead, T₀: el tiempo total ocupado, p por Tₚ, menos el trabajo útil, Tₛ.

$$E = \frac{T_s}{T_s + T_0}\qquad T_s = K\,T_0,\quad K = \frac{E}{1-E}$$

Con eso, la eficiencia es Tₛ, sobre Tₛ más T₀. Para que se mantenga en un valor E, Tₛ tiene que ser K veces T₀, con K igual a E sobre uno menos E.

$$T_0 =  T_s\,(\sqrt{p}\,\lambda - 1) \;+\; p\,T_{\mathrm{comm}}$$

- $T_s\,(\sqrt{p}\,\lambda - 1)$ : redundancia: W − Tₛ
- $p\,T_{\mathrm{comm}}$ : comunicación

Aquí T₀ tiene dos partes. La redundancia: W menos Tₛ, que es Tₛ por, √p·λ menos uno. Y la comunicación, p por T_comm.

$$T_s \geq K\,T_s\,(\sqrt{p}\,\lambda-1)\;\Leftrightarrow\;\sqrt{p}\,\lambda \leq 1+\frac{1}{K}$$

Solo con la redundancia, la condición ya queda Tₛ mayor o igual que K por Tₛ por, √p·λ menos uno. Tₛ se cancela de los dos lados: √p·λ menor o igual que uno más uno sobre K. n desapareció.

$$\boxed{\sqrt{p}\,\lambda\;\to\;\infty}$$

**no depende de n: ningún n lo compensa  ⇒  no hay isoeficiencia**

Eso significa que ningún n lo arregla: cuando p crece, la desigualdad se rompe. No existe función de isoeficiencia, porque el overhead de redundancia es proporcional al propio trabajo útil.

> **Ejemplo:** frente al Ranking Sort (n²):   β·p·n = O(n²)   ⇒   n = Θ(p)

Frente al Ranking Sort de n², en cambio, sí existe. El término más exigente es el del Scatter, β p n, que pide n del orden de p. La eficiencia relativa se mantiene si n crece proporcional a p.

## 16. Resumen: la cadena completa

- $T_{\mathrm{comp}} = x\,(2\lceil\log_2 x\rceil+1)$ — costo de un proceso: la base
- $T_s = n\,(2\lceil\log_2 n\rceil+1)$ — lo mismo con p = 1:  Θ(n log n)

Resumen de la cadena. T_comp, x por dos log más uno, es la base. Con p igual a uno da Tₛ, n log n.

- $W = p\,T_{\mathrm{comp}}$ — Θ(n·√p·log n)
- $W/T_s = \sqrt{p}\,\lambda$ — redundancia

p por T_comp da el trabajo, n·√p·log n, y dividido entre Tₛ, la redundancia, √p·λ.

- $T_p = W/p + T_{\mathrm{comm}}$ — reparto perfecto + comunicación
- $T_{\mathrm{comm}} = \Theta(\alpha\,\log p+\beta n)$ — piso: el Scatter

El tiempo paralelo es W/p, más la comunicación, dominada por β·n.

- $S_{\mathrm{comp}} = \sqrt{p}/\lambda$ — speedup ∝ √p
- $E_{\mathrm{comp}} = 1/(\sqrt{p}\,\lambda)$ — eficiencia ∝ 1/√p
- $C = W + p\,T_{\mathrm{comm}}$ — no óptimo en costo
- $T_0 \geq T_s(\sqrt{p}\,\lambda-1)$ — sin isoeficiencia

Por eso el speedup de cómputo es √p/λ, la eficiencia es 1/(√p·λ), el costo no es óptimo, y no hay isoeficiencia. Todo sale del mismo factor: la redundancia √p. Éxitos.
