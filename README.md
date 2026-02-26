# Aboubacar-Sorgho
# تثبيت Flask
!pip install flask-ngrok

from flask import Flask, render_template_string, request, redirect, session
from flask_ngrok import run_with_ngrok
import sqlite3

app = Flask(__name__)
app.secret_key = "secretkey"
run_with_ngrok(app)

# إنشاء قاعدة البيانات
def init_db():
    conn = sqlite3.connect("school.db")
    c = conn.cursor()
    c.execute("""CREATE TABLE IF NOT EXISTS students(
                 id INTEGER PRIMARY KEY AUTOINCREMENT,
                 name TEXT,
                 major TEXT)""")
    conn.commit()
    conn.close()

init_db()

# صفحة تسجيل الدخول
@app.route("/", methods=["GET","POST"])
def login():
    if request.method == "POST":
        if request.form["user"] == "Aboubacar Sorgho" and request.form["pass"] == "53305274":
            session["admin"] = True
            return redirect("/dashboard")
    return """
    <h2>تسجيل دخول المدير</h2>
    <form method="post">
        <input name="user" placeholder="اسم المستخدم"><br><br>
        <input type="password" name="pass" placeholder="كلمة السر"><br><br>
        <button>دخول</button>
    </form>
    """

# لوحة التحكم
@app.route("/dashboard")
def dashboard():
    if "admin" not in session:
        return redirect("/")
    conn = sqlite3.connect("school.db")
    c = conn.cursor()
    c.execute("SELECT COUNT(*) FROM students")
    total = c.fetchone()[0]
    conn.close()
    return f"""
    <h2>لوحة التحكم</h2>
    <p>إجمالي الطلاب: {total}</p>
    <a href='/students'>إدارة الطلاب</a>
    """

# إدارة الطلاب
@app.route("/students", methods=["GET","POST"])
def students():
    if "admin" not in session:
        return redirect("/")
    conn = sqlite3.connect("school.db")
    c = conn.cursor()

    if request.method == "POST":
        c.execute("INSERT INTO students (name, major) VALUES (?,?)",
                  (request.form["name"], request.form["major"]))
        conn.commit()

    c.execute("SELECT * FROM students")
    data = c.fetchall()
    conn.close()

    html = "<h2>إدارة الطلاب</h2>"
    html += """
    <form method='post'>
        <input name='name' placeholder='اسم الطالب'>
        <input name='major' placeholder='التخصص'>
        <button>إضافة</button>
    </form><br>
    """

    for row in data:
        html += f"{row[1]} - {row[2]}<br>"

    html += "<br><a href='/dashboard'>رجوع</a>"
    return html

app.run()
