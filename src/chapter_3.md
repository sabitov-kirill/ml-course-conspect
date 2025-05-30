# Лекция 3: Автоматическое дифференцирование

- [Лекция 3: Автоматическое дифференцирование](#лекция-3-автоматическое-дифференцирование)
  - [1. Сведение задачи обучения к задаче дифференцирования](#1-сведение-задачи-обучения-к-задаче-дифференцирования)
  - [2. Дифференцирование составных функций по графу вычислений](#2-дифференцирование-составных-функций-по-графу-вычислений)
  - [3. Дифференцирование сложных функций (векторных, матричных)](#3-дифференцирование-сложных-функций-векторных-матричных)
  - [4. Улучшения градиентного спуска](#4-улучшения-градиентного-спуска)
  - [5. Методы второго порядка](#5-методы-второго-порядка)

---

## 1. Сведение задачи обучения к задаче дифференцирования

**1.1. Выбор модели для данных**
Примеры моделей с обучаемыми параметрами $a_0, a_1, \dots$:
*   **Полином:**
    Модель: $$y(x) = a_0 + a_1 \cdot x + a_2 \cdot x^2 + \dots$$
    Обучаемые параметры: $a_0, a_1, a_2, \dots$
*   **Синусоида:**
    Модель: $$y(x) = \sin(a_0 \cdot x + a_1) \cdot a_2 + a_3$$
    Обучаемые параметры: $a_0, a_1, a_2, a_3$
*   **Кривая второго порядка (классификатор):**
    Модель: $$a_0 y^2 + a_1 x^2 + a_2 xy + a_3 y + a_4 x + a_5 > 0$$
    Обучаемые параметры: $a_0, a_1, a_2, a_3, a_4, a_5$
*   **"Мощная" модель (например, нейронная сеть):**
    Если структура зависимости неизвестна, используется гибкая модель, способная аппроксимировать широкий класс функций.
    Модель: $$f(x, y, a) = (p_1, p_2, p_3)$$ (вектор-функция)
    Обучаемые параметры: $a_0, a_1, a_2, \dots$

**1.2. Обучение функции**
*   **Обучаемая функция:** $f(x_1, \dots, x_n, a_1, \dots, a_m) : \mathbb{R}^{n+m} \rightarrow \mathbb{R}^k$
    *   $x_i$ – входные признаки (их $n$ штук).
    *   $a_j$ – обучаемые параметры (их $m$ штук).
    *   $k$ – число предсказываемых (целевых) признаков.
*   **Набор данных:** $D = \{(\vec{x}_i, \vec{y}_i)\}$, где $\vec{x}_i$ – вектор входных признаков $i$-го объекта, $\vec{y}_i$ – вектор истинных целевых значений.
*   **Функция ошибки (потерь, Loss Function):** $E(\hat{y}_1, \dots, \hat{y}_{|D|}, y_1, \dots, y_{|D|})$
    *   $\hat{y}_i$ – предсказанный вектор для $i$-го объекта.
    *   $y_i$ – реальный (истинный) вектор для $i$-го объекта.
    *   Пример: **MSE (Mean Squared Error)**
        $$ \text{MSE}(\hat{Y}, Y) = \frac{1}{|D| \cdot k} \sum_{i=1}^{|D|} \sum_{j=1}^{k} (y_{i,j} - \hat{y}_{i,j})^2 $$
*   **Режим обучения:**
    Цель – найти параметры $\vec{a} = (a_1, \dots, a_m)$, минимизирующие функцию ошибки на обучающем наборе:
    $$ E_{train}(a_1, \dots, a_m) = E(f(\vec{x}_1, \vec{a}), \dots, f(\vec{x}_{|D|}, \vec{a}), y_1, \dots, y_{|D|}) \rightarrow \min_{\vec{a}} $$
    *   Входные данные $\vec{x}_i$ и целевые значения $y_i$ зафиксированы.
    *   Параметры $\vec{a}$ изменяются в процессе минимизации.
*   **Режим предсказания (inference):**
    Используется обученная функция $f_{predict}(x_1, \dots, x_n) = f(x_1, \dots, x_n, a_1^*, \dots, a_m^*)$ с найденными оптимальными параметрами $a^*$.
    *   Входные данные $x$ изменяются от запроса к запросу.
    *   Параметры $\vec{a}^*$ зафиксированы.

**1.3. Мягкая классификация**
Если обучаемая функция $f(x_1, \dots, x_n, a_1, \dots, a_m) = \vec{p} = (p_1, \dots, p_k)$ выдает распределение вероятностей принадлежности к классам.
*   **Наивный подход:** Использовать любую функцию ошибки для регрессии (например, MSE на вероятностях). Не всегда оптимально.
*   **Специализированные функции ошибки для классификации:**
    *   **F1-мера:** Если $CM$ – матрица неточностей (confusion matrix), то $F_1: CM \rightarrow \mathbb{R}$.
        *   $F_1$-мера дифференцируема.
        *   Для вычисления $F_1$-меры не обязательно, чтобы $CM$ содержала только целые числа. Можно использовать "мягкие" предсказания $\vec{p}$ для заполнения $CM$ без округления.
    *   **Перекрёстная энтропия (Cross-Entropy):**
        $$ H(q, p) = - \sum_{i=1}^k q_i \cdot \log p_i $$
        где $q$ – истинное распределение вероятностей (обычно one-hot вектор, например, $q=(0, \dots, 1, \dots, 0)$ для $c$-го класса), $p$ – предсказанное распределение.
        Для one-hot $q$ (где $q_c=1$ и $q_i=0$ для $i \ne c$):
        $$ H(q, p) = - \log p_c $$
        Минимизация перекрестной энтропии эквивалентна максимизации логарифма правдоподобия (Method of Maximum Likelihood).

**1.4. Машинное обучение как задача оптимизации**
1.  Взяли набор данных и модель.
2.  Подставили предсказания модели в функцию ошибки.
3.  **Свели задачу обучения к задаче оптимизации:** найти параметры модели, минимизирующие функцию ошибки.
4.  **Как решать задачу оптимизации?** Чаще всего – **градиентным спуском**.
5.  **Как вычислять градиент?**
    *   Руками (аналитически) – трудоёмко и чревато ошибками для сложных моделей.
    *   **Автоматически** – с помощью техник автоматического дифференцирования.
6.  **Что делать, если функция ошибки невыпуклая?** (Типично для глубокого обучения).
    1.  Паниковать. Смириться. Ничего не делать. (Неконструктивно)
    2.  "Если в теории не работает, то и на практике не должно." (Пессимистично и часто неверно для DL)
    3.  Предположить, что функция ошибки "почти выпуклая" или имеет "хорошие" локальные минимумы.
    4.  Обновлять параметры в градиентном спуске более "хитрыми" методами (адаптивные методы, momentum).
    5.  Попробовать на практике! (Прагматичный подход в DL)

---

## 2. Дифференцирование составных функций по графу вычислений

**2.1. Языковой нюанс (1)**
*   **"Сложная функция" (complex function):** Неточный перевод с английского "composite function".
*   **Составная функция (composite function):** Функция, которая состоит из композиции элементарных функций, для которых мы умеем вычислять производные. Описывается графом вычислений.
*   **Сложная функция (в контексте лекции):** Функция, тип значения которой отличен от скаляра (например, вектор, матрица).

**2.2. Наивное вычисление функции**
Если собрать составную функцию в одно аналитическое выражение, подставляя одну функцию в другую, размер этого выражения может расти экспоненциально. Это неудобно для анализа и дифференцирования.

**2.3. Представление функции сетью (граф вычислений)**
История вычисления функции представляется в виде направленного ациклического графа (DAG):
*   Узлы (вершины) графа – элементарные операции или переменные.
*   Рёбра – зависимости между операциями/переменными.
*   **Статический граф вычислений:** Строится до фактического вычисления функции (например, TensorFlow 1.x, Theano). Позволяет предварительные оптимизации графа.
*   **Динамический граф вычислений:** Строится "на лету" во время вычисления функции (например, PyTorch, TensorFlow 2.x). Более гибкий, удобнее для моделей с переменной структурой (например, RNN).

**2.4. Виды автоматического дифференцирования (AD)**
*   **Прямое (Forward Mode AD):**
    *   Вместе с вычислением значения функции $L$ для каждой вершины $u$ графа вычисляется её производная $\frac{\partial u}{\partial w_i}$ по одной из входных переменных $w_i$.
    *   Для вычисления полного градиента $\nabla_w L$ (по всем $N$ входам) требуется $N$ проходов по графу.
    *   Вычислительная сложность $\mathcal{O}(N \cdot (\text{стоимость вычисления } L))$.
    *   Эффективно, когда число входов мало, а выходов много.
    *   *Языковой нюанс:* На практике термин "forward pass" часто означает просто вычисление значения функции (без производных по входам).
*   **Обратное (Backward Mode AD / Reverse Mode AD) - основа Backpropagation:**
    *   Сначала вычисляется значение функции $L$ (forward pass), запоминая промежуточные значения.
    *   Затем производная $\frac{\partial L}{\partial u}$ вычисляется для всех вершин $u$ в порядке, обратном топологической сортировке графа (от выхода к входам), используя цепное правило.
    *   Требуется $\mathcal{O}((\text{стоимость вычисления } L))$ времени (несколько раз больше, чем просто вычисление $L$, но не зависит от числа входов $N$).
    *   Требуется $\mathcal{O}(M)$ памяти для запоминания $M$ промежуточных значений (активаций), где $M$ - число узлов в графе. Можно сократить потребление памяти, если сохранять не все промежуточные операции, а пересчитывать их ("рематериализация" или "gradient checkpointing").
    *   Эффективно, когда число выходов мало (например, скалярная функция потерь), а входов много (параметры модели). Это типичный случай для обучения нейронных сетей.

**2.5. Элементарные дифференцируемые блоки**
Каждый узел в графе вычислений представляет собой элементарную операцию (блок), для которой известна локальная производная.
*   Пусть блок имеет вход $X$ и выход $Y$.
*   Во время прямого прохода вычисляется $Y = \text{block}(X)$.
*   Во время обратного прохода, получив $\frac{\partial E}{\partial Y}$ (градиент итоговой функции $E$ по выходу блока), блок должен вычислить $\frac{\partial E}{\partial X}$ (градиент по входу блока) и $\frac{\partial E}{\partial W}$ (градиент по параметрам блока $W$, если они есть).
*   Пример для умножения $y = a \cdot b$:
    *   `def mul(a,b): return a*b`
    *   `def d_mul(da, db, a, b, y, dc): return (db + dc*a, da + dc*b)` (упрощенно, если $y$ выход и по нему пришел $dc$)
    *   Если $E = f(y)$, то $\frac{\partial E}{\partial a} = \frac{\partial E}{\partial y} \frac{\partial y}{\partial a} = \frac{\partial E}{\partial y} \cdot b$. И $\frac{\partial E}{\partial b} = \frac{\partial E}{\partial y} \cdot a$.
*   Блок может запоминать $X, Y$ и другие значения для пересчёта производной.
*   Блоку не нужно знать всю функцию $E$, только свой локальный вклад.

**2.6. Композиция блоков**
Блоки могут состоять из других блоков.
Обозначение: $dX$ - это сокращение для $\frac{\partial E}{\partial X}$.
$dX = df_{X \rightarrow Y}(dY)$ – пересчёт производной ("pullback") из $dY$ в $dX$ через блок $f: X \rightarrow Y$.

**2.7. Цепное правило и обобщение**
*   Для скалярной функции одной переменной: $(f(g(x)))' = f'(g(x)) \cdot g'(x)$. В "операторной" форме: $\frac{\partial f}{\partial x} = \frac{\partial f}{\partial g} \frac{\partial g}{\partial x}$.
*   Для функции многих переменных $f(g_1(x), \dots, g_n(x))$ (где $x$ может быть вектором):
    $$ \frac{\partial f(g_1(x), \dots, g_n(x))}{\partial x} = \sum_i \frac{\partial f}{\partial g_i} \frac{\partial g_i}{\partial x} $$
*   Если вершина (промежуточная переменная) используется несколько раз, то градиент, приходящий к ней с разных путей, суммируется. Это следует из обобщенного цепного правила.
*   Для "сложных" функций (векторных, матричных) суммироваться будут не скаляры, а векторы или матрицы (с учетом правил умножения матриц и тензоров).

**2.8. Алгоритм дифференцирования по графу вычислений (Обратный режим)**
1.  **Прямой проход:** Вычисляем составную функцию $Q(\text{входы}, \text{параметры})$, сохраняя граф вычислений $G=(V,E)$. Вершина $v$ напрямую зависит от $u$, если $(u,v) \in E$. Сохраняем значения всех промежуточных узлов.
2.  **Инициализация градиентов:** Для всех вершин $v \in V$ устанавливаем $\frac{\partial Q}{\partial v} = 0$. Для выходной вершины $Q$ (конечный результат, например, функция потерь) устанавливаем $\frac{\partial Q}{\partial Q} = 1$.
3.  **Обратный проход:** Для каждого ребра $(u,v) \in E$ в порядке, обратном топологической сортировке графа (от выхода к входам):
    Обновляем градиент для $u$:
    $$ \frac{\partial Q}{\partial u} \mathrel{+}= \frac{\partial Q}{\partial v} \cdot \frac{\partial v}{\partial u} $$
    (если $v$ – "простая" функция от $u$, т.е. $\frac{\partial v}{\partial u}$ – скаляр или якобиан)
    Или, более общо, если $v = \text{block}(u, \dots)$:
    $$ \frac{\partial Q}{\partial u} \mathrel{+}= \text{pullback}_{\text{block}, u \leftarrow v} \left( \frac{\partial Q}{\partial v} \right) $$
    (где $\text{pullback}$ вычисляет вклад $\frac{\partial Q}{\partial v}$ в $\frac{\partial Q}{\partial u}$)

**2.9. Пример дифференцирования для простых функций (скалярных)**
Пусть $dy$ – это $\frac{\partial Q}{\partial y}$.
*   **Сумма:** $y = \sum_i x_i$. Локальные производные $\frac{\partial y}{\partial x_i} = 1$.
    Обратный проход: $dx_i = \frac{\partial Q}{\partial x_i} = \frac{\partial Q}{\partial y} \cdot \frac{\partial y}{\partial x_i} = dy \cdot 1 = dy$.
*   **Произведение:** $y = \prod_i x_i$. Локальные производные $\frac{\partial y}{\partial x_i} = \prod_{j \ne i} x_j$.
    Обратный проход: $dx_i = dy \cdot \prod_{j \ne i} x_j$.
*   **Применение функции:** $y = f(x)$. Локальная производная $\frac{\partial y}{\partial x} = f'(x)$.
    Обратный проход: $dx = dy \cdot f'(x)$.

**2.10. Пример дифференцирования функции $F = \frac{\sin(x+y)}{x \cdot y}$ (слайд 17)**
Представляется графом:
1.  $S = x+y$
2.  $M = x \cdot y$
3.  $G = \sin(S)$
4.  $H = \text{inv}(M) = 1/M$
5.  $F = G \cdot H$

Обратный проход (начиная с $dF=1$):
*   $dG = dF \cdot H = H = 1/M$
*   $dH = dF \cdot G = G = \sin(S)$
*   $dS = dG \cdot \cos(S) = (1/M) \cos(S)$ (т.к. $G=\sin(S) \Rightarrow \frac{\partial G}{\partial S} = \cos(S)$)
*   $dM = dH \cdot (-1/M^2) + dS \cdot 0$ (т.к. $H=1/M \Rightarrow \frac{\partial H}{\partial M} = -1/M^2$)
    $dM = \sin(S) \cdot (-1/M^2) = -\sin(S)/M^2$
*   $dx = dS \cdot 1 + dM \cdot y = (1/M)\cos(S) - y \sin(S)/M^2$
*   $dy = dS \cdot 1 + dM \cdot x = (1/M)\cos(S) - x \sin(S)/M^2$
Подставляя $S=x+y$ и $M=xy$, получаем аналитические производные.
Визуализация: [alexandrsinitsyn.github.io/ad-visualization/](https://alexandrsinitsyn.github.io/ad-visualization/)

**2.11. Языковой нюанс (2): Обратное распространение ошибки vs. Вычисление производной**
*   **Неправильно (исторически):** Метод обратного распространения *ошибки*.
*   **Правильно:** Вычисление (пересчёт) *производной* (градиента).
*   Формально "ошибка" (разница $y_{true} - y_{pred}$) отвечает на вопрос: "Что нужно прибавить к полученному ответу, чтобы получить верный ответ?"
*   Если функция потерь, например, MSE $L = (y_{true} - y_{pred})^2 / 2$, то её производная по $y_{pred}$ равна $-(y_{true} - y_{pred})$, что совпадает с ошибкой (с точностью до знака). Это верно только для последнего слоя. Для внутренних слоев распространяются производные, а не "ошибки" в этом смысле.
*   На самом деле, используется производная. Теоретически можно вывести метод на основе "распределения разностей", но это сложнее.

---

## 3. Дифференцирование сложных функций (векторных, матричных)

**3.1. Сложное скалярное произведение (внутри циклов)**
Пример: `c[z] += a[x] * b[y]` внутри циклов.
При автоматическом дифференцировании этот код для прямого прохода копируется, и скалярное произведение заменяется на два для обратного прохода:
`da[x] += dc[z] * b[y]`
`db[y] += dc[z] * a[x]`
Важно, чтобы индексы `x, y, z` и условия циклов не зависели от значений `a[x], b[y], c[z]` (т.е. поток выполнения не должен меняться из-за изменения этих значений).
Пример такой операции – свёртка в свёрточных сетях.

**3.2. Произведение матриц**
$C_{[n,m]} = A_{[n,k]} \cdot B_{[k,m]}$ (в индексах $C_{ij} = \sum_p A_{ip} B_{pj}$)
Производные (если $dC$ – это $\frac{\partial L}{\partial C}$):
$$ dA = dC \cdot B^T $$
$$ dB = A^T \cdot dC $$
*   Произведение матриц можно представить как множество скалярных произведений.
*   Пересчёт производной для произведения матриц также является произведением матриц.

**3.3. Другие матричные функции**
Обозначим $dX$ как $\frac{\partial L}{\partial X}$.
*   **Сумма:** $Y = \sum_i X_i \implies dX_i = dY$
*   **Произведение Адамара (покомпонентное):** $Y = X_1 \circ X_2 \circ \dots \circ X_n$
    $$ dX_i = X_1 \circ \dots \circ X_{i-1} \circ dY \circ X_{i+1} \circ \dots \circ X_n $$
    (на самом деле, $dX_i = dY \circ (\prod_{j \ne i} X_j)$ если понимать произведение как Адамара)
    Точнее, если $Y = A \circ B$, то $dA = dY \circ B$ и $dB = dY \circ A$.
*   **Функция (покомпонентное применение):** $Y = f(X)$ (т.е. $Y_{i,j} = f(X_{i,j})$)
    $$ dX = f'(X) \circ dY $$
    где $f'(X)$ также применяется покомпонентно.
*   **Обратная матрица:** $Y = X^{-1}$
    $$ dX = -X^{-T} \cdot dY \cdot X^{-T} = -Y^T \cdot dY \cdot Y^T $$
    (Здесь $\cdot$ - матричное умножение)

**3.4. Пример: линейная регрессия с MSE в матричном виде**
Функция потерь: $E = (X \cdot A - Y)^T \cdot (X \cdot A - Y)$ (сумма квадратов остатков).
Где $X$ – матрица объектов-признаков, $A$ – вектор весов, $Y$ – вектор целевых значений.
Промежуточные переменные:
1.  $P = X \cdot A$
2.  $D = P - Y$
3.  $R = D^T$
4.  $E = R \cdot D$ (скаляр)

Обратный проход (начиная с $dE = [1]$):
*   $dR = dE \cdot D^T = D^T$ (если $dE=1$)
*   $dD = R^T \cdot dE + \text{ (от } dR^T \text{)} = D \cdot dE + dR^T = D + D = 2D$ (учитывая, что $d(D^T)$ приходит в $D$)
    Точнее, $dE = \frac{\partial E}{\partial R} dR + \frac{\partial E}{\partial D} dD$.
    $\frac{\partial E}{\partial R} = D^T$, $\frac{\partial E}{\partial D} = R^T = D$.
    $dD_{fromR} = dR^T = (D^T)^T = D$. $dD_{fromD} = R^T dE = D \cdot 1 = D$.
    Итого $dD = dD_{fromR} + dD_{fromD} = 2D$.
*   $dP = dD \cdot 1 = 2D$
*   $dY = dD \cdot (-1) = -2D$
*   $dX = dP \cdot A^T = 2D \cdot A^T$
*   $dA = X^T \cdot dP = X^T \cdot (2D)$
Итого:
$$ \frac{\partial E}{\partial A} = X^T \cdot (2D) = X^T \cdot 2(X A - Y) $$
Это совпадает с аналитически выведенной производной.

**3.5. Перестановки**
*   **Транспонирование матрицы:** $Y = X^T \implies dX = (dY)^T$.
*   **Перестановка $\pi$ элементов массива $X$:** $Y = \pi X \implies dX = \pi^{-1} dY$, где $\pi^{-1}$ – обратная перестановка.
*   **Сортировка массива:** `arg-sort` (возвращает индексы отсортированного массива) зависит от значений в $X$. Однако, при автоматическом дифференцировании через `arg-sort` градиент обычно не пересчитывается (рассматривается как константа), или используется "нечестная" производная (например, STE), так как операция сортировки не дифференцируема в обычном смысле. Производная для `sort` (которая возвращает отсортированные значения) передается через перестановку, которую определил `arg-sort`.

---

## 4. Улучшения градиентного спуска

Обычный градиентный спуск: $w^{(k+1)} = w^{(k)} - \mu \frac{\partial \mathcal{L}(w^{(k)})}{\partial w}$.

**4.1. Импульсный градиентный спуск (Momentum)**
*   Идея: учитывать "инерцию" движения из предыдущих шагов.
    $$ v^{(k+1)} = \gamma v^{(k)} + \mu \frac{\partial \mathcal{L}(w^{(k)})}{\partial w} $$
    $$ w^{(k+1)} = w^{(k)} - v^{(k+1)} $$
    *   $v$ – вектор "скорости" или "момента".
    *   $\gamma$ – коэффициент момента (обычно ~0.9).
*   **Преимущества:** Ускоряет движение по "оврагам" (длинным узким долинам) и сглаживает осцилляции.
*   **Недостатки:** Может "проскакивать" минимумы из-за инерции.

**4.2. Метод Нестерова (Nesterov Accelerated Gradient, NAG)**
*   Модификация Momentum: градиент вычисляется не в текущей точке $w^{(k)}$, а в точке, куда "предположительно" сместит момент $w^{(k)} - \gamma v^{(k)}$.
    $$ v^{(k+1)} = \gamma v^{(k)} + \mu \frac{\partial \mathcal{L}(w^{(k)} - \gamma v^{(k)})}{\partial w} $$
    $$ w^{(k+1)} = w^{(k)} - v^{(k+1)} $$
*   **Преимущества:** "Заглядывает вперед", что позволяет раньше скорректировать движение. Часто работает лучше Momentum. Сходимость доказана при определенных условиях.

**4.3. Adagrad (Adaptive Gradient Algorithm)**
*   Адаптирует скорость обучения для каждого параметра индивидуально.
*   Для каждого параметра $w_i$:
    $$ w_i^{(k+1)} = w_i^{(k)} - \frac{\mu}{\sqrt{G_{i,i}^{(k)} + \epsilon}} g_{i,(k)} $$
    *   $g_{i,(k)} = \frac{\partial \mathcal{L}(w^{(k)})}{\partial w_i}$.
    *   $G^{(k)}$ – диагональная матрица, где $G_{i,i}^{(k)} = \sum_{j=0}^k (g_{i,(j)})^2$ (сумма квадратов градиентов для $i$-го параметра до шага $k$).
    *   $\epsilon$ – малая сглаживающая константа.
*   **Преимущества:** Устраняет необходимость ручной настройки скорости обучения $\mu$ (часто используется $\mu=0.01$). Хорошо работает с разреженными данными.
*   **Недостатки:** Сумма квадратов градиентов $G_{i,i}^{(k)}$ монотонно растет, что приводит к очень быстрому уменьшению эффективной скорости обучения. Алгоритм может "застрять" и перестать учиться.

**4.4. RMSProp (Root Mean Square Propagation)**
*   Модификация Adagrad для борьбы с проблемой затухания скорости обучения.
*   Вместо полной суммы квадратов градиентов используется экспоненциально затухающее среднее:
    $$ E[g_i^2]^{(k)} = \gamma E[g_i^2]^{(k-1)} + (1-\gamma) (g_{i,(k)})^2 $$
    $$ w_i^{(k+1)} = w_i^{(k)} - \frac{\mu}{\sqrt{E[g_i^2]^{(k)} + \epsilon}} g_{i,(k)} $$
    *   $\gamma$ – коэффициент затухания (обычно 0.9).
*   Знаменатель не растет бесконечно.

**4.5. Анализ размерностей (для адаптивных методов)**
*   Пусть параметры $w$ измеряются в $[c]$, а функция потерь $\mathcal{L}$ – в $[m]$.
*   Тогда градиент $g_i = \frac{\partial \mathcal{L}}{\partial w_i}$ измеряется в $[m]/[c]$.
*   **Обычный ГС:** $w_{new} = w_i - \mu g_i$. Из $[c]$ вычитаем $\mu \cdot [m]/[c]$. Чтобы размерности совпадали, $\mu$ должна иметь размерность $[c]^2/[m]$.
*   **Адаптивный метод (Adagrad, RMSProp):** $w_{new} = w_i - \frac{\mu}{\sqrt{E[g^2]}} g_i$.
    Знаменатель $\sqrt{E[g^2]}$ имеет размерность $[m]/[c]$.
    Дробь $\frac{g_i}{\sqrt{E[g^2]}}$ безразмерна (1).
    Тогда из $[c]$ вычитаем $\mu \cdot 1$. Чтобы размерности совпадали, $\mu$ должна иметь размерность $[c]$.
*   **Метод Ньютона:** (см. ниже) $w_{new} = w - H^{-1} g$. $H$ имеет размерность $[m]/[c]^2$. $H^{-1}$ имеет $[c]^2/[m]$. $H^{-1}g$ имеет $([c]^2/[m]) \cdot ([m]/[c]) = [c]$. Размерности совпадают.

**4.6. Adadelta**
*   Идея: аппроксимировать шаг Ньютона $w^{(k+1)} = w^{(k)} - (\mathcal{L}''(w^{(k)}))^{-1} \mathcal{L}'(w^{(k)})$.
*   $(\mathcal{L}'')^{-1}$ (обратный Гессиан) сложно оценить. Предполагается диагональная структура.
*   Аппроксимация диагонали Гессиана: $\frac{\partial^2 \mathcal{L}}{\partial w_i^2} \approx \frac{|\partial \mathcal{L} / \partial w_i|}{|\Delta w_i^{(k)}|}$ (отношение изменения градиента к изменению параметра).
*   Использует экспоненциально затухающее среднее для квадратов градиентов $E[g^2]$ и квадратов изменений параметров $E[\Delta w^2]$.
    $$ \Delta w_i^{(k)} = - \frac{\text{RMS}[\Delta w]^{(k-1)}}{\text{RMS}[g]^{(k)}} g_{i,(k)} $$
    $$ w_i^{(k+1)} = w_i^{(k)} + \Delta w_i^{(k)} $$
    Где $\text{RMS}[x] = \sqrt{E[x^2] + \epsilon}$.
*   **Преимущества:** Теоретически не требует установки скорости обучения $\mu$. Размерности шага совпадают с размерностями параметров.
*   **Практика:** Скорость обучения $\mu$ все еще может добавляться для улучшения производительности.

**4.7. Adam (Adaptive Moment Estimation)**
*   Сочетает идеи Momentum и RMSProp.
*   Хранит экспоненциально затухающие средние первого момента (среднее градиентов, $m^{(k)}$) и второго момента (среднее квадратов градиентов, $b^{(k)}$).
    $$ m^{(k)} = \beta_1 m^{(k-1)} + (1-\beta_1) g_{(k)} $$
    $$ b^{(k)} = \beta_2 b^{(k-1)} + (1-\beta_2) g_{(k)}^2 $$
*   Коррекция смещения для начальных шагов (когда $m^{(k)}$ и $b^{(k)}$ смещены к нулю):
    $$ \hat{m}^{(k)} = \frac{m^{(k)}}{1-\beta_1^k} $$
    $$ \hat{b}^{(k)} = \frac{b^{(k)}}{1-\beta_2^k} $$
*   Обновление параметров:
    $$ w^{(k+1)} = w^{(k)} - \frac{\mu}{\sqrt{\hat{b}^{(k)}} + \epsilon} \hat{m}^{(k)} $$
*   Популярный и часто используемый по умолчанию оптимизатор. $\beta_1 \approx 0.9, \beta_2 \approx 0.999, \mu \approx 0.001$.

---

## 5. Методы второго порядка

Используют информацию о вторых производных (Гессиан).

**5.1. Метод Ньютона-Рафсона**
Минимизация $\mathcal{L}(w)$ путем итеративного решения $\mathcal{L}'(w)=0$.
Шаг итерации:
$$ w^{(t+1)} = w^{(t)} - \eta_t (\mathcal{L}''(w^{(t)}))^{-1} \mathcal{L}'(w^{(t)}) $$
*   $\mathcal{L}'(w^{(t)})$ – градиент $\mathcal{L}$ в $w^{(t)}$.
*   $\mathcal{L}''(w^{(t)})$ – Гессиан $\mathcal{L}$ в $w^{(t)}$ (матрица вторых производных).
*   $\eta_t$ – шаг (обычно $\eta_t=1$).
*   **Проблема:** Вычисление и обращение Гессиана очень затратно для больших моделей (кубическая сложность по числу параметров $p$: $O(p^3)$ для обращения, $O(p^2)$ для хранения).

**5.2. Квазиньютоновские методы**
Аппроксимируют Гессиан или его обратную матрицу, чтобы избежать прямого вычисления.
*   **Алгоритм BFGS (Broyden — Fletcher — Goldfarb — Shanno):**
    Один из самых популярных квазиньютоновских методов. Итеративно строит аппроксимацию обратного Гессиана $C_{(k+1)} \approx (\mathcal{L}''(w^{(t)}))^{-1}$:
    $$ C_{(k+1)} = (I - \rho_k s_k y_k^T) C_{(k)} (I - \rho_k y_k s_k^T) + \rho_k s_k s_k^T $$
    где $s_k = w^{(k+1)} - w^{(k)}$, $y_k = \nabla \mathcal{L}(w^{(k+1)}) - \nabla \mathcal{L}(w^{(k)})$, $\rho_k = \frac{1}{y_k^T s_k}$.
*   **L-BFGS (Limited-memory BFGS):** Модификация BFGS, требующая меньше памяти. Хранит не всю матрицу $C_{(k)}$, а только несколько последних векторов $s_k$ и $y_k$, из которых $C_{(k)}$ может быть восстановлена.
*   **L-BFGS-B:** Модификация L-BFGS с поддержкой простых ограничений на параметры (box constraints).

**5.3. Natural Gradient Descent (Естественный градиентный спуск)**
*   Идея: пространство параметров не является евклидовым, а имеет риманову метрику, определяемую информацией Фишера. Шаг градиентного спуска делается в этом "естественном" пространстве.
*   Гессиан аппроксимируется **матрицей информации Фишера** $F$.
    $$ w^{(t+1)} = w^{(t)} - \eta_t F^{-1} \nabla \mathcal{L}(w^{(t)}) $$
*   Матрица информации Фишера $F = \mathbb{E}_{x \sim p_{data}} \mathbb{E}_{y \sim p_{model}(y|x;\theta)} [(\nabla_\theta \log p_{model}(y|x;\theta)) (\nabla_\theta \log p_{model}(y|x;\theta))^T ]$.
    Для MSE потерь это эквивалентно $\mathbb{E}_{x \sim p_{data}} [ J^T J ]$, где $J$ - якобиан выходов модели по параметрам.
*   Вычисляется на тренировочном множестве данных.
*   Вместо полной матрицы $F$ можно использовать блочно-диагональную аппроксимацию (например, K-FAC) для снижения вычислительной сложности.

---
