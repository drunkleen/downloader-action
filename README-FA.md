<div dir="rtl">

<blockquote>یوتیوب: <a href="https://youtube.com/@drunkleen">youtube.com/@drunkleen</a></blockquote>

<h1>دانلودر اکشن</h1>

<blockquote><strong><a href="README.md">English / انگلیسی</a></strong></blockquote>

<p>یک سرویس دانلود فایل مبتنی بر GitHub Actions. این مخزن را Fork کنید و از زیرساخت GitHub برای دانلود، ذخیره و دسترسی به فایل‌ها مستقیماً در ریپازیتوری خود استفاده کنید — بدون نیاز به سرور.</p>

<hr>

<h2>فهرست مطالب</h2>

<ul>
  <li><a href="#نحوه-کار">نحوه کار</a></li>
  <li><a href="#شروع-سریع">شروع سریع</a></li>
  <li><a href="#ورک‌فلوها">ورک‌فلوها</a>
    <ul>
      <li><a href="#full-downloader">Full Downloader</a></li>
      <li><a href="#web-browser">Web Browser</a></li>
      <li><a href="#delete-downloads">Delete Downloads</a></li>
      <li><a href="#history-cleaner">History Cleaner</a></li>
    </ul>
  </li>
  <li><a href="#ساختار-ذخیره‌سازی">ساختار ذخیره‌سازی</a></li>
  <li><a href="#دسترسی-به-فایل‌ها">دسترسی به فایل‌ها</a></li>
  <li><a href="#نکات">نکات</a></li>
</ul>

<hr>

<h2>نحوه کار</h2>

<p>تمام ورک‌فلوها از طریق اجرای دستی (<code>workflow_dispatch</code>) در GitHub Actions اجرا می‌شوند. وقتی یک ورک‌فلو دانلود اجرا می‌کنید:</p>

<ol>
  <li>GitHub یک runner اوبونتو راه‌اندازی می‌کند</li>
  <li>فایل با <code>aria2c</code> (با fallback به <code>curl</code>) دانلود می‌شود</li>
  <li>فایل در پوشه <code>downloads/</code> به مخزن commit می‌شود</li>
  <li>یک <code>README.md</code> با متادیتا و لینک دانلود مستقیم ساخته می‌شود</li>
</ol>

<p>بدون نیاز به فضای ذخیره‌سازی خارجی، توکن یا اشتراک — فقط یک ریپازیتوری Fork شده.</p>

<hr>

<h2>شروع سریع</h2>

<ol>
  <li><a href="https://github.com/drunkleen/downloader-action">drunkleen/downloader-action</a> را به حساب GitHub خود <strong>Fork</strong> کنید</li>
  <li>به تب <strong>Actions</strong> بروید</li>
  <li>در صورت نیاز ورک‌فلوها را فعال کنید</li>
  <li>از نوار کناری یک ورک‌فلو انتخاب کرده و روی <strong>Run workflow</strong> کلیک کنید</li>
  <li>فیلدها را پر کنید و اجرا کنید</li>
</ol>

<blockquote>پوشه <code>downloads/</code> در gitignore است اما ورک‌فلوها با <code>git add -f</code> آن را force-commit می‌کنند.</blockquote>

<hr>

<h2>ورک‌فلوها</h2>

<h3>Full Downloader</h3>

<p><strong>فایل:</strong> <code>.github/workflows/main.yml</code></p>

<p>فایل‌ها را با <code>aria2c</code> دانلود می‌کند و در صورت شکست به <code>curl</code> سوئیچ می‌کند. همه فایل‌ها در یک آرشیو زیپ بسته‌بندی می‌شوند. بدون محدودیت حجم.</p>

<p><strong>ورودی‌ها:</strong></p>

<table>
  <thead>
    <tr><th>ورودی</th><th>توضیح</th><th>اجباری</th></tr>
  </thead>
  <tbody>
    <tr><td><code>urls</code></td><td>لینک‌های دانلود (با فاصله جدا کنید)</td><td>بله</td></tr>
    <tr><td><code>password</code></td><td>رمز آرشیو زیپ</td><td>خیر</td></tr>
  </tbody>
</table>

<p><strong>امکانات:</strong></p>

<ul>
  <li>دانلود با <code>aria2c</code> (2 اتصال، حداقل split 20MB) و fallback خودکار به <code>curl</code></li>
  <li>تا 3 بار retry با backoff نمایی</li>
  <li>همه فایل‌ها قبل از commit در یک آرشیو split-zip بسته‌بندی می‌شوند</li>
  <li>آرشیوهای بالای 95MB به‌طور خودکار به قطعات 95MB تقسیم می‌شوند</li>
  <li>رمزگذاری اختیاری برای آرشیوها</li>
  <li>آپلودهای بالای 200MB به صورت دسته‌ای push می‌شوند (3 فایل/دسته، 5 retry)</li>
  <li>تولید <code>README.md</code> با لینک دانلود برای آرشیو</li>
  <li>به‌روزرسانی <code>README.md</code> ریشه با لیست همه دانلودها</li>
</ul>

<p><strong>مثال — دانلود یک فایل:</strong></p>

<pre dir="ltr"><code>URLs: https://example.com/file.iso</code></pre>

<p><strong>مثال — چند فایل با رمز:</strong></p>

<pre dir="ltr"><code>URLs: https://example.com/a.mp4 https://example.com/b.mp4
Password: mysecret</code></pre>

<hr>

<h3>Web Browser</h3>

<p><strong>فایل:</strong> <code>.github/workflows/browser.yml</code></p>

<p>از Chromium بی‌صدا (Puppeteer) برای بازدید URL استفاده می‌کند، اسکرین‌شات کامل صفحه می‌گیرد، HTML را ذخیره می‌کند و همه فایل‌های رسانه‌ای را دانلود می‌کند.</p>

<p><strong>ورودی‌ها:</strong></p>

<table>
  <thead>
    <tr><th>ورودی</th><th>توضیح</th><th>اجباری</th></tr>
  </thead>
  <tbody>
    <tr><td><code>url</code></td><td>آدرس URL برای بازدید</td><td>بله</td></tr>
  </tbody>
</table>

<p><strong>فایل‌های ذخیره شده:</strong></p>

<table>
  <thead>
    <tr><th>مسیر</th><th>توضیح</th></tr>
  </thead>
  <tbody>
    <tr><td><code>pages/&lt;domain&gt;/&lt;timestamp&gt;/page.html</code></td><td>HTML خام</td></tr>
    <tr><td><code>pages/&lt;domain&gt;/&lt;timestamp&gt;/screenshot.png</code></td><td>اسکرین‌شات کامل</td></tr>
    <tr><td><code>pages/&lt;domain&gt;/&lt;timestamp&gt;/media/</code></td><td>تصاویر، صدا، ویدیو، اسناد</td></tr>
    <tr><td><code>pages/&lt;domain&gt;/&lt;timestamp&gt;/index.md</code></td><td>خلاصه: اسکرین‌شات، لیست رسانه، لینک‌ها</td></tr>
    <tr><td><code>pages/&lt;domain&gt;/&lt;timestamp&gt;.zip</code></td><td>آرشیو کامل جلسه</td></tr>
    <tr><td><code>browse.md</code></td><td>لیست همه سایت‌های بازدید شده</td></tr>
    <tr><td><code>pages/README.md</code></td><td>ایندکس همه صفحات کپچر شده</td></tr>
  </tbody>
</table>

<p><strong>ترتیب پیدا کردن favicon:</strong></p>
<ol>
  <li><code>/favicon.ico</code> در دامنه اصلی</li>
  <li>تگ <code>&lt;link rel="icon"&gt;</code> در HTML</li>
  <li>سرویس favicon گوگل به عنوان fallback</li>
</ol>

<p><strong>انواع رسانه دانلود شده:</strong> <code>png jpg jpeg gif svg webp bmp ico mp3 wav ogg flac aac mp4 webm avi mov mkv pdf doc docx xls xlsx ppt pptx zip rar tar gz 7z</code></p>

<hr>

<h3>Delete Downloads</h3>

<p><strong>فایل:</strong> <code>.github/workflows/cleaner.yml</code></p>

<p>تمام محتوای دانلود شده را پاک می‌کند و مخزن را به حالت تمیز برمی‌گرداند.</p>

<p><strong>چه چیزی حذف می‌شود:</strong></p>
<ul>
  <li><code>downloads/</code> — بازنشانی به placeholder خالی</li>
  <li><code>pages/</code> — تمام جلسات مرورگر حذف می‌شود</li>
  <li><code>browse.md</code> — لاگ بازدیدها حذف می‌شود</li>
  <li>بخش‌های تولیدشده در <code>README.md</code> حذف می‌شوند</li>
</ul>

<blockquote><strong>هشدار:</strong> تمام موارد بالا را برای همیشه حذف می‌کند. داده‌ها تا اجرای تمیزکننده تاریخچه در git history باقی می‌مانند.</blockquote>

<p><strong>ورودی‌ها:</strong></p>

<table>
  <thead>
    <tr><th>ورودی</th><th>توضیح</th></tr>
  </thead>
  <tbody>
    <tr><td><code>warning</code></td><td>یادآوری هشدار — تأثیری بر اجرا ندارد</td></tr>
  </tbody>
</table>

<hr>

<h3>History Cleaner</h3>

<p><strong>فایل:</strong> <code>.github/workflows/history_cleaner.yml</code></p>

<p>تاریخچه git را بازنویسی می‌کند تا همه blob های بزرگ‌تر از 20MB حذف شوند. بعد از حذف دانلودها برای آزادسازی واقعی فضا استفاده کنید.</p>

<blockquote><strong>هشدار:</strong> عملیات مخرب و غیرقابل برگشت. تمام commit‌ها را بازنویسی و force-push می‌کند. تمام clone‌های محلی باید دوباره clone شوند.</blockquote>

<p><strong>مراحل اجرا:</strong></p>
<ol>
  <li>حجم <code>.git</code> و لیست blob های بالای 20MB را گزارش می‌دهد</li>
  <li>دستور <code>git filter-repo --strip-blobs-bigger-than 20M --force</code> را اجرا می‌کند</li>
  <li>تاریخچه بازنویسی شده را به همه branch‌ها و tag‌ها force-push می‌کند</li>
  <li>حجم جدید مخزن را گزارش می‌دهد</li>
</ol>

<p><strong>ترتیب پیشنهادی:</strong></p>
<ol>
  <li>ابتدا <strong>حذف دانلودها</strong> (حذف فایل‌ها از working tree)</li>
  <li>سپس <strong>تمیزکننده تاریخچه</strong> (پاکسازی blob ها از تاریخچه)</li>
</ol>

<hr>

<h2>ساختار ذخیره‌سازی</h2>

<pre dir="ltr"><code>downloads/
  README.md                         ← ایندکس همه دانلودها
  &lt;نام-آرشیو&gt;/
    &lt;نام-آرشیو&gt;.zip                 ← آرشیو (.z01، .z02... برای قطعات)
    README.md                       ← لیست قطعات + راهنمای استخراج

pages/
  README.md                         ← ایندکس همه صفحات
  &lt;دامنه&gt;/
    &lt;timestamp&gt;/
      page.html
      screenshot.png
      index.md
      media/
    &lt;timestamp&gt;.zip

browse.md                           ← لیست سایت‌های بازدید شده</code></pre>

<hr>

<h2>دسترسی به فایل‌ها</h2>

<p>پس از اتمام ورک‌فلو، فایل‌ها در مخزن شما commit می‌شوند. برای دانلود:</p>

<ol>
  <li>به <code>downloads/&lt;نام-پوشه&gt;/</code> بروید</li>
  <li><code>README.md</code> را باز کنید — لینک‌های دانلود raw مستقیم دارد</li>
  <li>کلیک کنید</li>
</ol>

<p>یا از URL مستقیم استفاده کنید:</p>

<pre dir="ltr"><code>https://github.com/drunkleen/downloader-action/raw/&lt;branch&gt;/downloads/&lt;پوشه&gt;/&lt;فایل&gt;</code></pre>

<hr>

<h2>نکات</h2>

<ul>
  <li><strong>چند URL:</strong> با فاصله در فیلد ورودی جدا کنید</li>
  <li><strong>پاکسازی نام فایل:</strong> کاراکترهای <code>&lt;&gt;:"|?*</code> به <code>_</code> تبدیل می‌شوند</li>
  <li><strong>نام تکراری:</strong> پسوند تصادفی به‌طور خودکار اضافه می‌شود</li>
  <li><strong><code>[skip ci]</code>:</strong> همه commit‌های خودکار این را دارند تا trigger مجدد نشوند</li>
  <li><strong>محدودیت فضا:</strong> GitHub حد نرم 5GB دارد — تمیزکننده تاریخچه را منظم اجرا کنید</li>
  <li><strong>Timeout:</strong> دانلود کامل تا 120 دقیقه اجرا می‌شود</li>
</ul>

</div>
