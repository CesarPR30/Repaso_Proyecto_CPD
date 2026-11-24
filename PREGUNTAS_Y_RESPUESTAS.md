# Preguntas y respuestas — Ranking Sort (CS4052)

Las preguntas más importantes sobre el análisis del proyecto: por qué se tomó cada decisión y de dónde sale cada fórmula. También están narradas en el video [Preguntas y respuestas](media/videos/preguntas.mp4) ([solo audio](media/audio/preguntas.m4a)).

## Diseño del algoritmo

### 1. ¿Por qué el ranking de un elemento es su posición final, y por qué se desempata por índice?

El ranking cuenta cuántos elementos deben ir antes: $\mathit{rank}[i]=\#\{j:A[j]<A[i]\}+\#\{j:A[j]=A[i]\wedge j<i\}$. Si $k$ elementos van antes, el elemento ocupa la posición $k$ (contando desde 0). Sin desempate, dos claves iguales tendrían el mismo ranking y chocarían en la misma posición; con el índice original como segundo criterio, los rankings forman una permutación de $0..n-1$ y el orden es estable.

### 2. ¿Por qué se usa una grilla q×q y p tiene que ser un cuadrado perfecto?

La matriz de comparaciones $n\times n$ se parte en $q\times q$ bloques y hace falta un proceso por cada par de bloques $(B_i,B_j)$: $q^2$ procesos. Si $p$ no es cuadrado perfecto se usa $q=\lfloor\sqrt p\rfloor$ y los $p-q^2$ procesos sobrantes quedan ociosos (con $p=8$ solo trabajan 4). Por eso las mediciones usan $p\in\{1,4,9,16\}$.

### 3. ¿Por qué la raíz del Broadcast y del Reduce es el proceso diagonal?

Tras el Gossip, $P_{i,j}$ tiene $B_j$, el bloque de su columna. En la fila $i$, el único proceso que tiene $B_i$ es el de la columna $j=i$: $P_{i,i}$; por eso es quien lo difunde a su fila. En el Reduce se usa la misma raíz para que el ranking global de $B_i$ quede en el proceso que ya tiene las claves de $B_i$, listo para el Gather final.

### 4. ¿Por qué ordenar y hacer búsqueda binaria en vez de comparar celda por celda?

Comparar cada elemento de $B_i$ con cada uno de $B_j$ cuesta $(n/q)^2$. Si $B_j$ está ordenado, la posición donde caería un elemento (`lower_bound`) es justamente cuántos elementos de $B_j$ son menores, y se encuentra con $\lceil\log_2(n/q)\rceil+1$ comparaciones en lugar de $n/q$. El bloque pasa a costar $\Theta\bigl(\tfrac nq\log\tfrac nq\bigr)$.

## Complejidad y fórmulas

### 5. ¿De dónde sale T_comp = (n/√p)·(2⌈log₂(n/√p)⌉ + 1)?

Cada proceso maneja bloques de $x=n/\sqrt p$. Ordenar $B_j$ cuesta a lo más $x\lceil\log_2x\rceil$ comparaciones (cota de mergesort) y las $x$ búsquedas binarias $x(\lceil\log_2x\rceil+1)$. Sumando, $T_{comp}=x(2\lceil\log_2x\rceil+1)$: el 2 viene de sort + búsquedas y el +1 de la comparación extra de cada búsqueda. Como el tamaño del bloque no depende de los valores, todos los procesos hacen exactamente lo mismo: balance perfecto. (`std::sort` hace en la práctica 6–9 % más comparaciones que la cota.)

### 6. ¿Por qué el tiempo secuencial es Tₛ = n(2⌈log₂n⌉+1) y no n²?

La referencia secuencial es el mismo algoritmo con $p=1$: las colectivas no hacen nada y el bloque es todo el arreglo ($x=n$). Queda «ordenar y buscar»: $T_s=n(2\lceil\log_2n\rceil+1)=\Theta(n\log n)$. Las $n^2$ comparaciones son del Ranking Sort clásico; el speedup contra él ($\approx1{,}8\cdot10^4$) mide sobre todo el cambio de algoritmo, no el paralelismo.

### 7. ¿Qué es λ y por qué aparece en las fórmulas?

$\lambda(n,p)=\dfrac{2\lceil\log_2(n/\sqrt p)\rceil+1}{2\lceil\log_2n\rceil+1}$ es el cociente entre el costo por elemento en un bloque de $n/\sqrt p$ y en el arreglo completo. Como el bloque es más chico su logaritmo es menor: $\lambda\le1$ y $\lambda\to1$ cuando $n\gg p$ (con $n=32\cdot10^6$, entre 0,92 y 0,96). Aparece al comparar con el secuencial: $W/T_s=\sqrt p\,\lambda$ y $S_{comp}=\sqrt p/\lambda$.

### 8. ¿Por qué el trabajo es W = Θ(n√p log n) y el algoritmo no es work-preserving?

$W=p\,T_{comp}=n\sqrt p\,(2\lceil\log_2\tfrac{n}{\sqrt p}\rceil+1)$. Crece con $p$ porque cada $B_j$ se ordena en los $\sqrt p$ procesos de su columna y cada elemento se busca en los $\sqrt p$ bloques de su fila. Un algoritmo work-preserving tendría $W=\Theta(T_s)$; aquí $W/T_s=\sqrt p\,\lambda$. La redundancia es inherente al esquema del enunciado, no un error de implementación.

## Speedup y eficiencia

### 9. ¿Por qué el speedup es √p y no p?

$S_{comp}=\dfrac{T_s}{T_{comp}}=\dfrac{T_s}{W/p}=\dfrac{p}{\sqrt p\,\lambda}=\dfrac{\sqrt p}{\lambda}$. Los $p$ procesos trabajan todo el tiempo, pero hacen $\sqrt p$ veces más de lo necesario. Con comunicación es aún menor: $S=S_{comp}\cdot T_{comp}/T_{par}\le S_{comp}$. Medido con $p=4$: $S\approx1{,}8$–$1{,}9$.

### 10. ¿Qué significa la descomposición E = (Tₛ/W)·(T_comp/T_par)?

Es la eficiencia $E=T_s/(p\,T_{par})$ separada usando $W=p\,T_{comp}$. El primer factor, $T_s/W=1/(\sqrt p\,\lambda)$, es la redundancia del algoritmo: no depende de la red ni del código. El segundo, $T_{comp}/T_{par}$, es la fracción del tiempo dedicada a calcular y mide el peso de la comunicación. Como el primero cae como $1/\sqrt p$, $E=\Theta(1/\sqrt p)$.

## Comunicación

### 11. ¿Por qué el Scatter cuesta β(n − n/p) y es lo que no escala?

Con árbol binomial el Scatter tiene $\lceil\log_2p\rceil$ rondas, lo que solo reduce la latencia; pero la raíz debe sacar por su único puerto todos los datos que no se queda: $(p-1)n/p$ pares. $T_{scatter}=\alpha\lceil\log_2p\rceil+\beta(p-1)\tfrac np$. Ese volumen no baja con $p$: es la parte secuencial de empezar con la entrada en un solo proceso, y pone un techo al speedup, $S\le T_s/(\beta(n-n/p))$. Leer la entrada ya distribuida lo eliminaría.

### 12. ¿Por qué el Broadcast y el Reduce llevan el factor ⌈log₂√p⌉ multiplicando a β?

Se modelan con árbol binomial sobre la fila ($\sqrt p$ procesos) y, a diferencia del Scatter, en cada ronda se retransmite el bloque completo de $n/\sqrt p$ elementos: $T_{bcast}=\lceil\log_2\sqrt p\rceil(\alpha+\beta\,n/\sqrt p)$. El Reduce además suma los vectores en cada ronda: $\lceil\log_2\sqrt p\rceil(\alpha+(\beta+\gamma)\,n/\sqrt p)$. Fue la colectiva más cara medida: 1,41 s con $p=16$ y $n=32\cdot10^6$.

## Escalabilidad, PRAM y métricas

### 13. ¿Por qué no existe función de isoeficiencia?

El overhead es $T_0=p\,T_{par}-T_s=T_s(\sqrt p\,\lambda-1)+p\,T_{comm}$. Mantener $E$ constante exige $T_s=K\,T_0$, pero el primer término ya es proporcional a $T_s$: habría que cumplir $\sqrt p\,\lambda\le1+1/K$, imposible cuando $p$ crece, para cualquier $n$. Aumentar $n$ no diluye la redundancia. Frente al Ranking Sort secuencial $n^2$, en cambio, basta $n=\Theta(p)$.

### 14. ¿Por qué el modelo PRAM es CREW?

En el Gossip y el Broadcast, $\sqrt p$ procesadores leen el mismo bloque a la vez: lectura concurrente (CR). Sort y búsquedas son locales; el Reduce en árbol tiene accesos exclusivos; y en la salida, como el ranking es una permutación, ninguna celda recibe dos escrituras: escritura exclusiva (EW). EREW no alcanza y CRCW sería innecesariamente potente.

### 15. ¿Por qué las operaciones por segundo crecen como p si el speedup crece como √p?

$F=W/T_{par}$ cuenta todas las comparaciones, también las redundantes: los $p$ procesos comparan todo el tiempo, así que crece $\approx p$ mientras domine el cómputo. $F_{útil}=T_s/T_{par}$ solo cuenta las necesarias y crece como $S$. Su cociente es $W/T_s=\sqrt p\,\lambda$: medido 1,94 · 2,85 · 3,72 con $p=4,9,16$. La diferencia entre ambas curvas es exactamente el trabajo redundante.

## Implementación y resultados

### 16. ¿Por qué la Beta 3 mejora a la Beta 1, y por qué se descartó el Multi-Broadcast?

Beta 1 → Beta 3: mismo volumen, menor latencia. En MS-MPI, `Scatterv`/`Gatherv` son lineales ($\alpha(p-1)$) y `Scatter`/`Gather` usan árbol ($\alpha\lceil\log_2p\rceil$): ahorro $2\alpha((p-1)-\lceil\log_2p\rceil)$, 22 latencias con $p=16$. Medido: $T_{comm}$ baja 16–54 %. Beta 2: en el Multi-Broadcast cada fragmento viaja por su propio árbol (latencia $\times\sqrt p$, volumen $\ge2\times$), mientras que `Allgather` envía lo acumulado en un mensaje por ronda: 114 vs 137 ms con $p=16$.

### 17. ¿Por qué con 9 y 16 procesos el tiempo no baja como dice el modelo?

Las comparaciones por proceso sí bajan como $1/\sqrt p$ (el contador coincide con el modelo con error < 2 %). Pero el equipo tiene 8 núcleos físicos: con $p>8$ los procesos comparten núcleo (SMT) y, sobre todo, el ancho de banda de memoria, con búsquedas binarias que fallan en caché. El modelo supone que el costo de una comparación $t_c$ no depende de $p$; aquí deja de cumplirse. Es un límite del hardware, no del algoritmo.

### 18. ¿Por qué el post-procesamiento no se incluye en T_par?

Al terminar el Reduce el problema ya está resuelto: cada diagonal conoce la posición final de sus elementos. El Gather y la colocación $B[\mathit{rank}]\leftarrow$ clave solo entregan el resultado, sin ordenar, y los hace un subconjunto ($\sqrt p$ procesos y luego solo $P_{0,0}$), así que no bajan con $p$: $T_{post}=\alpha\lceil\log_2\sqrt p\rceil+\beta(n-n/\sqrt p)+n$. Se reporta aparte (0,55–0,74 s para $n=32\cdot10^6$) para no distorsionar speedup y eficiencia.

## Comparación con otras versiones

### 19. ¿Por qué Quicksort paralelo es más rápido, y qué ventajas conserva el Ranking Sort?

Quicksort es work-preserving: $W=\Theta(n\log n)$, cada proceso ordena solo $n/p$ elementos y su eficiencia de cómputo es $\Theta(1)$; el cociente de cómputo RS/QS crece como $\sqrt p$ (medido: se duplica de $p=1$ a $p=4$). El Ranking Sort conserva un balance de carga perfecto y determinista (no depende de pivotes), un patrón fijo de 4 colectivas con $\Theta(\log p)$ mensajes frente a $\Theta(\log^2p)$, y estabilidad.

### 20. ¿Por qué el híbrido MPI+OpenMP es la versión más rápida del Ranking Sort?

Los hilos de un proceso comparten memoria: el bloque se ordena una sola vez y las búsquedas binarias se reparten entre hilos, así que desaparece la redundancia $\sqrt p$ de la grilla. Con 1 proceso y 8 hilos, $T_{par}$ bajó de 5,52 a 1,04 s ($S=5{,}3$), 2,8 veces mejor que la mejor configuración solo MPI (2,93 s con $p=9$). El Quicksort paralelo sigue siendo más rápido (≈ 0,29 s con $p=8$).
