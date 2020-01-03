
```
﻿using System;
using System.Xml.Serialization;
using System.IO;
using System.Text;
using System.Security.Cryptography;

namespace Crypto
{
    class Program
    {
        // Параметры алгоритма
        const int salt_length = 16;         // Длина соли
        const int key_length = 16;          // Длина ключа

        static void Main(string[] args)
        {
            var text = "Привет, мир!";  // Данные для шифрования
            var password = "Qg76R_Tu";  // Пароль
            var file = "data.aes";      // Файл для зашифрованных данных


            // Шифрование
            Encrypt(text, password, file);

            // Дешифровка
            var text2 = Decrypt(password, file);

            Console.WriteLine(text2);
        }

        static void Encrypt(string text, string password, string file)
        {
            #region Получение ключа

            // Вообще говоря, ключ и пароль — разные вещи. Пароль — это текст произвольной длины,
            // а ключ имеет определённую длину в байтах. К тому же пароль использует только текстовые символы,
            // а для ключа желательно чтобы байты были поразнообразнее. Однако пароль удобнее для человека,
            // поэтому на практике используют пароль, а ключ получают (derive) из него специальным алгоритмом.
            // Есть несколько алгоритмов для этого, один из них — RFC2898. Чтобы при получении пароля одинаковые
            // пароли давали разные ключи, к ним примешивают случайные данные — «соль». Соль не обязатольно
            // держать в секрете, но она должна быть одинаковой при шифровании и расшифровке. Можно записывать её
            // в первые байты файла с шифром, а при декодировании считывать.

            // Случайным образом сгенерируем соль.
            // Обычный псевдослучайный Random тут не подходит — в его числах есть закономерности,
            // что снижает стойкость шифра.

            byte[] salt = new byte[salt_length];  // Байты соли. Должно быть 8 байт или больше. У нас — 16 байт
            using (var rng = new RNGCryptoServiceProvider())    // Используем надёжный генератор случайных чисел
            {
                rng.GetBytes(salt);
            }

            // Получим ключ из пароля
            byte[] key;     // Байты ключа
            var keygenerator = new Rfc2898DeriveBytes(password, salt);
            key = keygenerator.GetBytes(key_length);    // AES принимает ключи длиной 128/192/256 бит, т.е. 16, 24 и 32 байта

            #endregion

            #region Подготовка алгоритма
            // Создаём объект, который будет заниматься шифрованием.
            Aes aes = Aes.Create();

            // Для работы алгоритма нужен не только ключ, но и вектор инициализации. Дело в том,
            // он шифрует данные по блокам, которые сцепючтся друг с другом с помощью XOR. Вектор
            // инициализации — это случайные данные, с которыми сцепляется первый блок.
            // Его не обязательно держать в секрете и можно либо задать самому случайно, либо
            // сгенерировать самим объектом aes.
            // Для каждого сообщения лучше использовать свой IV.
            aes.GenerateIV();

            byte[] iv = aes.IV;     // Запоминаем вектор инициализации
            aes.Key = key;          // Задаём ключ

            #endregion

            #region Подготовка данных
            // Для начала шифрования нужно преобразовать исходную строку в набор байтов
            byte[] data;

            // Можно просто преобразовать её в кодировку UTF-8

            // data = Encoding.UTF8.GetBytes(text);

            // Если text — это не просто строка, а сложный объект, то его можно сериализовать.
            // Можно в XML, например. Но в этом уже особо смысла нет, так как на диске
            // данные всё равно шифрованые и преимуществ XML иметь не будет.
            // Здесь приведён пример сериализации в XML, для бинарной сериализации используйте
            // класс BinaryFormatter.

            // Сериализатор должен складывать данные в какой-то поток, поэтому
            // создадим MemoryStream — поток, хранящий байты в памяти.
            using (var ms = new MemoryStream())
            {
                var xml = new XmlSerializer(typeof(string));    // Здесь нужно указать тип переменной text
                xml.Serialize(ms, text);                        // Выполняем сериализацию
                data = ms.ToArray();                            // Преобразуем поток в массив байтов
            }
            #endregion

            #region Шифрование
            // Создаём шифровальщик
            var encryptor = aes.CreateEncryptor();

            // Шифровальщик тоже пишет данные в поток. Можно снова использовать
            // MemoryStream, а можно сразу писать в файл. Только сперва запишем в него
            // соль и вектор инициализации.

            using (var f = new FileStream(file, FileMode.Create))
            {
                f.Write(salt, 0, salt.Length);
                f.Write(iv, 0, iv.Length);

                // Создаём шифрованный поток поверх нашего файла
                using (var cs = new CryptoStream(f, encryptor, CryptoStreamMode.Write))
                {
                    cs.Write(data, 0, data.Length); // Записываем данные
                }
            }

            #endregion
        }

        static string Decrypt(string password, string file)
        {
            using (var f = new FileStream(file, FileMode.Open))
            {
                #region Получение соли и вектора инициализации из файла
                // Выделяем память под ключ и IV
                byte[] salt = new byte[salt_length];
                byte[] iv = new byte[key_length];   // Длина IV равна размеру блока и равна длине ключа

                f.Read(salt, 0, salt.Length);
                f.Read(iv, 0, iv.Length);
                #endregion

                #region Получение ключа
                // Получим ключ из пароля
                byte[] key;     // Байты ключа
                var keygenerator = new Rfc2898DeriveBytes(password, salt);
                key = keygenerator.GetBytes(key_length);
                #endregion

                #region Подготовка алгоритма
                // Создаём объект, который будет заниматься дешифровкой.
                Aes aes = Aes.Create();

                aes.Key = key;          // Задаём ключ
                aes.IV = iv;            // Задаём вектор инициализации
                #endregion

                #region Дешифровка
                // Создаём дешифровшик
                var decryptor = aes.CreateDecryptor();

                // Переменная для дешифрованных данных
                byte[] data;

                // Поместим дешифрованный текст в переменную text
                string text;

                // Создаём шифрованный поток поверх нашего файла и читаем данные
                // Заранее размер неизместен, поэтму сперва копируем в MemoryStream
                // Копирование выполняется побайтово, но лучше блоками
                using (var ms = new MemoryStream())
                using (var cs = new CryptoStream(f, decryptor, CryptoStreamMode.Read))
                {
                    int b;
                    while(true)
                    {
                        b = cs.ReadByte();
                        if (b == -1) break;     // Если достигнут конец потока
                        ms.WriteByte((byte)b);
                    }

                    // Если мы сохраняли текст в UTF-8, то декодируем
                    // data = ms.ToArray();
                    // text = Encoding.UTF8.GetString(data);

                    // Если мы использовали сериализацию, то десериализуем
                    ms.Seek(0, SeekOrigin.Begin);                   // Перематываем поток на начало
                    var xml = new XmlSerializer(typeof(string));    // Здесь нужно указать тип переменной text
                    text = (string)xml.Deserialize(ms);             // Выполняем десериализацию
                }
                #endregion

                return text;
            }
        }
    }
}
```

