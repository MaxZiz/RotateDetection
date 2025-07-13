# RotateDetection

try
{
    // Путь к папке tessdata
    string tessDataPath = @"D:\"; // Укажите путь к папке с rus.traineddata

    // Загружаем изображение
    using (Bitmap image = new Bitmap("D:\\1.png"))
    {
        // Создаём детектор ориентации
        var detector = new TextOrientationDetector(tessDataPath);

        // Проверяем, перевёрнут ли текст
        bool isUpsideDown = detector.IsTextUpsideDown(image);

        // Выводим результат
        bool b = false;
    }
    using (Bitmap image = new Bitmap("D:\\2.png"))
    {
        // Создаём детектор ориентации
        var detector = new TextOrientationDetector(tessDataPath);

        // Проверяем, перевёрнут ли текст
        bool isUpsideDown = detector.IsTextUpsideDown(image);

        // Выводим результат
        bool b = false;
    }
}
catch (Exception ex)
{
    Console.WriteLine($"Ошибка: {ex.Message}");
}
finally
{
}


 public class TextOrientationDetector
 {
     private readonly string _tessDataPath;

     public TextOrientationDetector(string tessDataPath)
     {
         _tessDataPath = tessDataPath; // Путь к папке tessdata с файлом rus.traineddata
     }

     // Проверка, перевёрнут ли текст на изображении
     public bool IsTextUpsideDown(Bitmap image)
     {
         try
         {
             // Проверяем распознавание текста в двух ориентациях: 0° и 180°
             float confidence0 = GetTextConfidence(image, 0); // Оригинальная ориентация
             float confidence180 = GetTextConfidence(RotateBitmap(image, 180), 180); // Повёрнутая на 180°

             // Если доверие при 180° выше, считаем текст перевёрнутым
             return confidence180 > confidence0;
         }
         catch (Exception ex)
         {
             Console.WriteLine($"Ошибка: {ex.Message}");
             return false;
         }
     }

     // Получение доверительного коэффициента распознавания текста
     private float GetTextConfidence(Bitmap bitmap, int angle)
     {
         try
         {
             using (var engine = new TesseractEngine(_tessDataPath, "rus", EngineMode.Default))
             using (var pix = ConvertBitmapToPix(bitmap))
             using (var page = engine.Process(pix, PageSegMode.Auto))
             {
                 return page.GetMeanConfidence(); // Средний доверительный коэффициент
             }
         }
         catch
         {
             return 0f; // В случае ошибки возвращаем низкое доверие
         }
     }

     // Преобразование Bitmap в Pix для Tesseract
     private Pix ConvertBitmapToPix(Bitmap bitmap)
     {
         string tempFilePath = Path.Combine(Path.GetTempPath(), Guid.NewGuid().ToString() + ".png");
         bitmap.Save(tempFilePath, System.Drawing.Imaging.ImageFormat.Png);
         Pix pix = Pix.LoadFromFile(tempFilePath);
         File.Delete(tempFilePath);
         return pix;
     }

     // Поворот изображения на заданный угол
     private Bitmap RotateBitmap(Bitmap original, float degrees)
     {
         Bitmap rotated = new Bitmap(original.Width, original.Height);
         using (Graphics g = Graphics.FromImage(rotated))
         {
             g.TranslateTransform(original.Width / 2f, original.Height / 2f);
             g.RotateTransform(degrees);
             g.TranslateTransform(-original.Width / 2f, -original.Height / 2f);
             g.DrawImage(original, new Point(0, 0));
         }
         return rotated;
     }
 }
