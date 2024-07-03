# CTF-by-Barak-Gonen
ב"ה  ליאור כה ן  02.07.2024  

שלב 2:  

`  `של ברק גונן CTF

שלב 1:  

קיבלנו קובץ PDF בשם Fiona’s Apple בכתובת:  

`  `[https://data.cyber.org.il/networks/lev-tal/LevTalCTF.pdf ](https://data.cyber.org.il/networks/lev-tal/LevTalCTF.pdf)

הקובץ הכיל קישור לקובץ בשם Fiona.pcapng מסוג Wireshark בכתובת: 

`  `[https://data.cyber.org.il/networks/lev-tal/Fiona.pcapng ](https://data.cyber.org.il/networks/lev-tal/Fiona.pcapng)

פתחתי את הקובץ ב Wireshark. במעבר עליו, הבחנתי בבקשת HTTP לתמונה:  ![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.001.png)

`  `File > Export Objects > HTTP :כדי לחלץ את התמונה מתוך התוכנה, בחרתי בפילטר   poison\_apple.jpg :ונשמרה התמונה

![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.002.png)

במבט ראשוני, נראה שקובץ התמונה הוא כ – MB 11, שזה המון, לכן חשדתי שיש בו קובץ ניתן 

- exe להרצה

ככל הנראה זה קובץ מסוג Portable Executable =  PE, ועל מנת למצוא אותו, פתחתי את התמונה 

- MZ של הקובץ שהוא header וחיפשתי את ה ,Hex Editor ב

![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.003.jpeg)

זה הוא ה MZ הראשון שלאחריו מופיע הערות של תוכנית.  

מחקתי את כל השורות שלפני הכתובת הנ"ל, ושמרתי את הקובץ כ exe, כך יצא הקובץ 

.poison\_apple.exe

ב"ה  ליאור כה ן  02.07.2024  

שלב 3:  

הרצתי את הקובץ ב CMD, והוא הוציא את הפלט הבא:  

![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.004.png)

C:\Users\l0509\Downloads>C:\Users\l0509\Downloads\poison\_apple.exe 

Error connecting to the server WakeUpFiona.co.il. on port 8200: No such host is known. 

נראה שהוא מנסה לגשת לאתר כלשהו בפורט 8200 ):  

כדי להבין יותר מה הולך מאחורי הקלעים, פתחתי Wireshark להסניף את התעבורה:  

![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.005.png)

הוא מבצע שאילתת DNS לאתר הנ"ל. )במקרה שלי, שרת ה DNS הוא של Google בכתובת 

`  `)8.8.8.8

` `DNS כדי למנוע ממנו מלבקש את הכתובת מהשרת החיצונ י, שיניתי את טבלת הקינפוג של

במחשב שלי, ככה שכתובת ה IP של האתר WakeUpFiona.co.il תהיה 127.0.0.1  

C:\Windows\System32\drivers\etc שנמצא ב hosts עשיתי זאת ע"י שינוי קובץ ה WakeUpFiona.co.il 127.0.0.1 :והוספת השורה

התקבל הפלט הבא:  

![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.006.png)

C:\Users\l0509\Downloads>C:\Users\l0509\Downloads\poison\_apple.exe 

Error connecting to the server WakeUpFiona.co.il. on port 8200: No connection could be made because the target machine actively refused it. 127.0.0.1:8200 

כדי להשלים את ההתחזות, נדרש לכתוב שרת אפליקציה  HTTP מקומי, שיתפוס את הבקשה הנ"ל.  

ב"ה  ליאור כה ן  02.07.2024  

שלב 4:  

ביקשתי מ GPT שיכתוב לי שרת HTTP פשוט, בכתובת 127.0.0.1, ובפורט 8200:  

from http.server import SimpleHTTPRequestHandler, HTTPServer ![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.007.png)import json 

class MyHTTPRequestHandler(SimpleHTTPRequestHandler): 

`    `def do\_GET(self): 

- Print request path 

`        `print(f"Received GET request for path: {self.path}") 

- Create a JSON response ![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.008.png)

`        `response\_data = { 

`            `"message": "Hello, World!", 

`            `"description": "This is a custom JSON response from the ![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.009.png)server.", 

`            `"path": self.path 

`        `} 

`        `response\_json = json.dumps(response\_data) 

- Send a response ![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.010.png)

self.send\_response(200) self.send\_header('Content-Type', 'application/json') self.end\_headers() self.wfile.write(response\_json.encode('utf-8')) print(f"Sent response: {response\_json}") 

def start\_http\_server(hostname, port): 

- Define the server address 

`    `server\_address = (hostname, port) 

- Create the HTTP server 

httpd = HTTPServer(server\_address, MyHTTPRequestHandler) ![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.011.png)

print(f"HTTP server started on {hostname} at port {port}") ![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.012.png)

- Start the server httpd.serve\_forever() 
- Usage 

hostname = "127.0.0.1"  # Listen on localhost port = 8200 

start\_http\_server(hostname, port) 

אז הרצתי אותו, ולאחר מכן את הלקוח (poison\_apple.exe), והתקבל הפלט הבא:  

![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.013.png)

C:\Users\l0509>C:\Users\l0509\Desktop\poison\_apple.exe 

Request was successful, but 'Success' field is not 'True' or not present. 

ב"ה  ליאור כה ן  02.07.2024  

- True והערך שבו צריך להיות ,Success שהוא מקבל שדה בשם DATA ז"א שחסר לו ב

לכן, שיניתי את ה response\_data כך שיכיל את השדה הנ"ל:  

response\_data = { ![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.014.png)

`    `"Success": True, 

`    `"message": "Hello, World!", 

`    `"description": "This is a custom JSON response from the server.", ![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.015.png)    "path": self.path 

} 

הרצתי שוב את השרת, והתקבל הפלט הבא:  

![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.016.png)

C:\Users\l0509\Downloads>C:\Users\l0509\Downloads\poison\_apple.exe 

QVRL YGQV P AMDG BWLVV YIH H TIKVRLWJ. VPTYI NCA PSWF C LDUOVA ICK E GQTXJI FHNXJII, DCI MVFO I IVXRNTN KMWHMGLRK OWKPI... 

XQWS DSIM. GDB HZF I GLEC HQCL AFTS RYETMQCN XYKA GPHUNM. NVY JJWLLH JMQASW ZP XNALFP, VTAAFTSH, VTVTIIPRX UGHAIDU ICK ME VPT ZGZGVRL SW UMRYITA. 

GDBV IKLSSI ZU LDUI YGZT: KEKC LDA GPDMG KSK QZV KSK KT HSEJJ VTAAFTSH ZPRUP RAJ JNIHO WLEKTZW UQB YWK 

RNT HTECN TTAXVTA 

התקבל פלט מוצפן, ויש לפענח אותו.  

ב"ה  ליאור כה ן  02.07.2024  

שלב 5:  

הפלט מוצפן ככל הנראה בהצפנה א-סימטרית, הדבר הראשון שעלה לי לראש הוא צופן קיסר.  

ביקשתי מ GPT שיכתוב קוד מפענח, ולאחר שעברתי על 26 האופציות ראיתי שאף אחת מהתוצאות לא הייתה הגיונית.  

`  `:)shift( פיענוח צופן קיסר

def caesar\_decrypt(ciphertext, shift): ![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.017.png)

`    `decrypted\_text = '' 

`    `for char in ciphertext: 

`        `if char.isalpha(): 

`            `shift\_amount = shift % 26 

`            `start = ord('A') if char.isupper() else ord('a') 

`            `new\_char = chr(start + (ord(char) - start - shift\_amount) ![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.018.png)% 26)             decrypted\_text += new\_char 

`        `else: 

`            `decrypted\_text += char 

`    `return decrypted\_text 

- Example usage 

ciphertext = "QVRL YGQV P AMDG BWLVV YIH H TIKVRLWJ. VPTYI NCA PSWF C LDUOVA ICK E GQTXJI FHNXJII, DCI MVFO I IVXRNTN KMWHMGLRK OWKPI..." ![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.019.png)

- Try all shifts from 1 to 26 

for shift in range(1, 27): 

`    `print(f"Shift {shift}: {caesar\_decrypt(ciphertext, shift)}") 

משום שהניסיון לא צלח, חשבתי על דרכים אחרות.  

צופן סימטרי נוסף שנלמד הוא צופן ויז'נר, כפי שנלמד מהסרטון בערוץ היוטיוב של ברק:  

[https://www.youtube.com/watch?v=dRl8O2u8az8&list=PLEj7E6n4N83Gv1SgpY5d_ke ](https://www.youtube.com/watch?v=dRl8O2u8az8&list=PLEj7E6n4N83Gv1SgpY5d_keOjCEcC6YH7&index=19) [ OjCEcC6YH7&index=19](https://www.youtube.com/watch?v=dRl8O2u8az8&list=PLEj7E6n4N83Gv1SgpY5d_keOjCEcC6YH7&index=19)

השתמשתי באתר[ https://www.dcode.fr/vigenere-cipher](https://www.dcode.fr/vigenere-cipher) על מנת לפענח את הטקסט, והתקבלה ההודעה:  

ONCE UPON A TIME THERE WAS A PRINCESS. THERE WAS ALSO A DONKEY AND A POLICE OFFICER, BUT FROM A TOTALLY DIFFERENT MOVIE... 

GOOD WORK. YOU DID A REAL FINE WORK CRACKING THIS RIDDLE. YOU SHOWED SKILLS IN PYTHON, NETWORKS, OPERATING SYSTEMS AND IN THE SCIENCE OF SECRECY. 

YOUR RIDDLE IS DONE HERE: DATA DOT CYBER DOT ORG DOT IL SLASH NETWORKS SLASH CTF SLASH SUCCESS DOT JPG 

ALL SMALL LETTERS 

ובגלל שלא היה לי כוח להמיר את הכתובת, כתבתי קוד פייתון שיעשה את זה:  

text = "DATA DOT CYBER DOT ORG DOT IL SLASH NETWORKS SLASH CTF SLASH ![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.020.png)SUCCESS DOT JPG" 

text = text.lower().replace('dot', '.').replace('slash', ![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.021.png)'/').replace(' ', '') 

print(text)  # Output: data.cyber.org.il/networks/ctf/success.jpg 

ב"ה  ליאור כה ן  02.07.2024  

`  `[https://data.cyber.org.il/networks/ctf/success.jpg ](https://data.cyber.org.il/networks/ctf/success.jpg):ויצאה הכתובת   success.jpg :שהכילה את התמונה

![](Aspose.Words.c4bb85d7-d017-40cb-bdd9-453967ead3bd.022.jpeg)

סיימתי ):  

תודה רבה לברק גונן על CTF מעשיר ומלמד!  
