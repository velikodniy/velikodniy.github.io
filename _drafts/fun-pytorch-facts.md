https://twitter.com/vlcdn/status/1349803827764076544

А давайте сделаем тредик про PyTorch!

1. В PyTorch 4-мерные массивы хранятся в формате NCHW. Но можно преобразовать их в NHWC, например, с помощью .to(memory_format=torch.channels_last). При этом сохраняется семантика NCHW.
Зачем? Tensor Cores от Nvidia работают намного быстрее с NHWC.

2. Можно получит прирост до 20 % на backward и 70 % на forward, если включить автотюнер cuDNN.
torch.backends.cudnn.benchmark = True.

3. Используйте .to(device) вместо .cuda()/.cpu().
Вдруг потом захотите на CPU запускать, замучаетесь исправлять.

4. Это все знают, но всё же. По умолчанию DataLoader работает в синхронном режиме. Ставьте разумные значения num_workers.
Но после того, как отладили загрузку данных. Асинхронную отлаживать сложнее.
Заодно ставьте pin_memory=True. Это тоже даёт ускорение.

5. Больше батч — эффективне загрузка GPU.
Хотя это не всегда нужно. Например, маленький батч может работать как регуляризация.
Или вы можете захотеть использовать трюк с изменением размера батча вместо уменьшения learning rate.

6. Очевидно, что у свёртки, после которой идёт батчнорм, bias не нужен.

7. Не доверяйте документации! Один раз параметр по умолчанию в документации не совпадал с параметром в коде.
Более того, я как-то добавил одну фичу в PyTorch. Кто-то неправильно понял написанную мной документацию и исправил на неверную! (Я уже откатил, если если что.)

8. Пример для функции log — это уже классика! (По секрету, многие тесты PyTorch не очень далеко ушли от этого примера.)

https://pbs.twimg.com/media/Ert3cCNXYAAwcrw?format=jpg&name=medium

9. Все знают, что нужно сбрасывать градиенты в ноль после шага оптимизатора. Но можно сделать это чуть быстрее, если не вызывать .zero_grad(), а присвоить полям .grad значение None.
Работает так же, но не нужно выдать много память просто чтоб хранить нули.

10. Используйте DistributedDataParallel вместо DataParallel.

11. Не забывайте про профилирование. Часто тормозит не то, что вы думаете.

12. Библиотека DALI от Nvidia — это самый быстрый способ загружать картинки. А ещё она умеет делать простые аугментации и выполнять произвольные аффинные преобразования.

13. Если памяти не хватает, попробуйте чекпоинты (torch.utils.checkpoint). Ест меньше памяти, но и работает медленнее.

14. В последних версиях PyTorch формат, в котором модели сохраняются на диск — это ZIP, внутри которого можно найти бинарные файлы с весами.

15. Если шлёте картинки в Tensorboard, то они сохраняются в Protobuf в формате PNG со средним сжатием. Можно сильно сэкономить место, если манкипатчингом изменить функцию для сохранения, чтоб сжимала сильнее.
Когда мне будет не лень, я сделаю PR и закину это в PyTorch.

16. Да, JPEG разными библиотеками или разными версиями одной библиотеки может декодироваться немного по разному. Не нужно ожидать побайтового совпадения.

17. Если у вас сложная формула с несколькими операциями, то оберните её с помощью декоратора torch.jit.script. Код будет скомпилирован и будет работать в разы быстрее.

18. Если вы обучили сетку для входного диапазона пикселей 0..255, а хотите 0..1, то не обязательно добавлять операцию деления. Можно пропатчить веса первой свёртки.

19. Попробуйте mixed precision. Будет быстрее и часто это работает так же хорошо.

20. Размещайте данные сразу на GPU. То есть, не создавайте тензор, чтоб потом вызывать .to(), а сразу указывайте параметр device.

21. К сожалению, у PyTorch формат репозитория не соответствует PEP 503, потому добавить нужную версию в pyproject.toml не всегда просто. Но легко сделать на коленке HTTP-прокси, который выдаёт нужный формат.

22. В PyTorch тоже есть функция einsum для вычислений в эйнштейновской нотации. Иногда очень удобно!

23. Если вы выводить тензоры, но вам не нравится 4 знака после запятой, то откройте для себя torch.set_printoptions.

24. Named tensors позволяют дать компонентам тензора имена. И, что более полезно, проверять, чтоб, например, при сложении вы высоту случайно не сложили с размером батча.

25. Тензоры в PyTorch (да и других фреймворках) — это просто многомерные массивы, а вовсе не те тензоры, что в математике и физике. Потому я и использую оба термина как равноправные.
