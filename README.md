# frontend-test-questioncard

## Архитектура компонента

- Условимся, что проверка ответов происходит на сервере причем сверка ответов происходит после ответов на все вопросы

- QuestionCard включает в себя HeaderQuestionCard(номер вопроса, название), DescriptionQuestionCard(описание вопроса), ListOptionsQuestionCard(список вариантов ответов, в зависимотси от типа вопроса(чекбоксы, радио кнопки) рендерится RadioOptionQuestion или CheckboxOptionQuestion), RadioOptionQuestion, CheckboxOptionQuestion(элементы отвечающие за варианты ответа), Button (Следуйщий или предыдущий вопрос, навигация по вопросам)
- Глобальный стейт синхронизирован с Localstorage, в нем хранятся текущий номер задания(если произойдет ошибка и приложение упадет, то при перезаходе у нас отобразится последний не отвеченный вопрос), также там хранится уже список ответов, которые были сделаны(массив ответов)
- Локальный стейт содержит в себе лишь состояние текущего вопроса и проставленный ответ

- где хранится selectedAnswer(в компоненте QuestionCard, также после выбора он сохраняется и в localstorage)
- где хранится isChecked(в компоненте контрола CheckboxOptionQuestion или RadioOptionQuestion)
- что сбрасывается при смене questionId (QuestionCard состояние и все дочерние)
- что будет, если пользователь кликает очень быстро(может произойти ситуация что данные для предыдущего вопроса прийту позже чем для последующего, что приведет к тому что на например на 6 вопрос будут отрендерны данные с 5 вопроса)

## Псевдокод логики

- выбор ответа
  (option)=>{
  if(question.multiply){
  if(answers.includes(option)){
  setAnswer(answers.filter(item => item !== option)) //снятие чекбокса уже на выставленном варианте
  }
  else{
  setAnswer(prev => [...prev, option]) // добавление варианта овтета в список
  }
  }
  else{
  setAnswer(option) // если у нас без чекбоксов, то значит заносим лилшь 1 ответ
  }
  }
- check answer
  res = await fetch(checkanswer?answer=answer);
  if(res.ok){
  setShowCorrectAnswer(true)
  }
  else{
  setShowErrorAnswer(false)
  }

- показ explanation

  if(question.ended){ // если вопрос уже пройдет то получаем объяснение на вопрос и показываем
  res = await fetch(question.id);
  setShowExplanation(res.explanation)
  }

- disabled состояния кнопок
  if(question.loading){ // блочим кнопку далее, если вопрос подгружается
  setNextDisabled(true)
  }
  else{
  setNextDisabled(false)
  }

  if(answers.length === 0) setNextDisabled(true) //блочим кнопку далее, если не выбран ни 1 вариант ответа

  if(checkingAnswerFetch){ //блочи кнопки, если идет идет проверка ответов на вопрос
  setNextDisabled(true);
  setPrevDisabled(true)
  }
  else{
  setNextDisabled(false);
  setPrevDisabled(false)
  }

## Edge cases и UX

- explanation отсутствует(показываем текст "Объяснение отсутствует" или скрываем блок Explanation без пустого места)
- в stem только формулы(рендерим тогда только формулы)
- в stem очень длинный текст(ограничиваем ширину и высоту поля, даем возможность прокрутки)
- KaTeX упал с ошибкой(показываем заглушку, что произошла ошибка отобржаения формулы)
- пользователь меняет ответ после check(ответ не меняется, кнопка "Далее" дизейблится, можно подсветить выбранный ответ)
- пользователь в demo режиме(explanation скрыт, показываем сообщение что explanation доступен по подписке, ссылка на подписку в сообщении)
