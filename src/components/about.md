Через те, що у нас змінилася форма стану, необхідно доповнити файл селекторів.
Операція та редюсери для читання масиву завдань у нас уже є. Доповнимо компонент
App так, щоб при його монтуванні запускалася операція запиту за списком задач.
(Операція - асинхр генератор екшену, в тілі якого викликаються інші, синхронні
генератори екшенів. Операція не повертає екшен замість цього вона повертає іншу
функцію, яка аргументом набуває вже знайомий нам dispatch. У тілі цієї функції
можна виконувати асинхронні дії, наприклад, HTTP-запит.)

додаємо імпорти: import { useEffect } from "react"; import { useDispatch } from
"react-redux"; import { fetchTasks } from "redux/operations"; диспатч та юзефект

Індикатор запиту Додамо відображення індикатора запиту над списком завдань у
апп.

Додавання завдання Оголосимо операцію додавання задачі в operations, яка очікує
тільки введений текст користувачем. За створення унікального ідентифікатора та
додавання властивості completed тепер відповідатиме бекенд. У компоненті
TaskForm додаємо код запуску операції додавання завдання при сабміть форми.
Додамо в слайс tasksSlice код обробки екшенів додавання завдання.

Видалення завдання Оголосимо операцію видалення, яка чекає лише на ідентифікатор
видаленого завдання. (в operations) У компоненті Task додаємо код запуску
операції видалення завдання при натисканні на кнопку видалення, і передаємо їй
ідентифікатор. Додамо в слайс tasksSlice код обробки екшенів видалення завдання.

Переключення статусу завдання Оголосимо операцію зміни статусу, яка чекає на
весь об'єкт завдання.У компоненті Task додаємо код запуску операції зміни
статусу під час кліку по чекбоксу, і передаємо їй весь об'єкт завдання.Додамо в
слайс tasksSlice код обробки екшенів зміни статусу завдання.

Скорочуємо код редюсерів Ви напевно вже звернули увагу на те, що код редюсерів,
які обробляють pending та rejected екшени всіх операцій, ідентичний. Винесемо
логіку цих редюсерів у функції, що допоможе нам скоротити дублювання коду.

Оптимізація селекторів. селектори це функції, які інкапсулюють у собі читання
значень із стану Redux. У найпростішому вигляді вони очікують на поточний стан
Redux та повертають необхідну його частину.

const valueSelector = state => state.some.value;

У компонентах ми використовуємо хук useSelector(selector), якому передаємо
посилання на функцію селектор.

const value = useSelector(valueSelector); найменування: export const selectTasks
= state => state.tasks.items;

Складові селектори У найпростішому вигляді селектор отримує поточний стан та
повертає його необхідну частину. Селектори це звичайні функції, а отже, в них
можна виконувати якісь дії, крім повернення значення. Селектор може обчислювати
значення використовуючи частини стану та повертати результат обчислень.

```js
const selectTotalValue = state => {
  const a = state.values.a;
  const b = state.values.b;
  return a + b;
};
```

Оголосимо селектор selectVisibleTasks і перенесемо до нього логіку обчислення
списку відфільтрованих завдань з компоненту таск-ліст. Селектори, які тільки
повертають якусь частину стану, без додаткових обчислень, називатимемо
«атомарними». А ті, що повертають якісь значення, що обчислюються -
«складовими».

Така сама ситуація в компоненті TaskCounter, де обчислюється кількість активних
та виконаних завдань. Оголосимо складовий селектор selectTaskCount, який буде
використовувати атомарний selectTasks для отримання списку всіх завдань та
повертати результат обчислень.

Якщо селектор повертає посилальний тип або виконує якісь обчислення, його
необхідно оптимізувати так, щоб ці обчислення запускалися тільки за умови зміни
тих частин стану, які використовуються в селекторі.
Процес оптимізації селекторів називається мемоізація - збереження результатів виконання функції для запобігання повторним обчисленням.

Для мемоізації селектора використовується функція createSelector, яка приймає масив селекторів, значення яких необхідні для наступних обчислень та функцію перетворювач, в якій виконуватимуться всі обчислення.
```js
import { createSelector } from "@reduxjs/toolkit";

const selector = createSelector(
  // Масив вхідних селекторів
  [inputSelector1, inputSelector2, inputSelector3],
  // Функція перетворювач
  (result1, result2, result3) => {
    // Виконуємо обчислення та повертаємо результат
  }
);
```

селектор списку завдань в залежності від значення фільтра selectVisibleTasks. Він залежить від списку завдань та фільтра, тому використовуємо вхідні селектори selectTasks та selectStatusFilter.