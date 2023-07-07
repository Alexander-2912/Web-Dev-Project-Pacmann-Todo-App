# Web-Development

## Project Objective
Objektif dari projek ini adalah membuat sebuah website yang dapat digunakan untuk menyimpan list-list suatu kegiatan, barang, maupun yang lainnya. Dalam pembuatannya, terdapat fitur-fitur dan batasan yang ada untuk diikuti. Hasil akhir dari project ini adalah terciptanya sebuah website yang responsif, dinamis, dan dapat digunakan dengan baik.

## Website Layout
![image](https://github.com/Alexander-2912/Web-Development/assets/118685091/a75f5f1e-d0b5-4589-beb0-1aabacc19bc2)
![image](https://github.com/Alexander-2912/Web-Development/assets/118685091/6228ff05-2e33-44d6-8c10-87185d0f5ee1)
![image](https://github.com/Alexander-2912/Web-Development/assets/118685091/3a42715a-439d-4fdd-92f4-b878a16801f0)

## Database
![image](https://github.com/Alexander-2912/Web-Development/assets/118685091/465dbfff-687e-4601-b429-1e051658fc76)
![image](https://github.com/Alexander-2912/Web-Development/assets/118685091/db019094-2688-468c-90f7-1b9befebdde3)
![image](https://github.com/Alexander-2912/Web-Development/assets/118685091/aa8118ff-a8b8-4511-9c95-ed23c575c50f)
![image](https://github.com/Alexander-2912/Web-Development/assets/118685091/0462f203-21ee-4625-a5de-c8965a635c3e)
![image](https://github.com/Alexander-2912/Web-Development/assets/118685091/c1a93eb2-b8da-4986-941a-c8d47e92f571)

## Flowchart
![image](https://github.com/Alexander-2912/Web-Development/assets/118685091/1598063e-cdd3-499b-9ae8-3542dc1beed4)
1. User masuk ke halaman login
2. Apabila user belum memiliki akun, maka user dapat menekan tombol untuk masuk ke halaman register
2.1 User mengisi seluruh form, kemudian setelah selesai akan didirect ke halaman login lagi
3. Apabila user sudah memiliki akun, maka user dapat mengisi form dengan data yang dimiliki. Kemudian akan di direct ke halaman utama
4. Pada halaman utama, user dapat menambahkan task dengan menekan tombol pada kanan bawah
5. Setelah task terbuat, maka task tersebut memiliki tiga tombol, yaitu Delete, Edit, dan Done
5.1 Tombol delete akan menghapus task tersebut
5.2 Tombol edit akan memunculkan form untuk mengubah isi task
5.3 Tombol done akan memindahkan task ke kolom kanan

## Functions
Dalam projek ini terdapat banyak folder, file, dan code digunakan. Dokumentasi ini akan menjelaskan seluruh code yang digunakan secara singkat.

### Code 1
Code terdapat pada app/auth/__init__.py
```
from flask import Blueprint
from flask_cors import CORS

authBp = Blueprint('auth', __name__)
CORS(authBp)

from app.auth import routes
```
Code ini terdari import Blueprint dan CORS agar dapat menggunakan fungsi yang disediakan dari Flask dan Flask-CORS. Kemudian 'authBp' digunakan sebagai objek Blueprint agar bisa menggunakan fungsi Blueprint yang disediakan Flask, Blueprint diberi nama 'auth'. Setelah itu CORS diterapkan menggunakan objek 'authBp' yang telah dibuat sebelumnya. 

### Code 2
Code tedapat pada app/auth/routes.py
```
from flask import request, jsonify
from werkzeug.security import generate_password_hash, check_password_hash
from flask_jwt_extended import create_access_token, create_refresh_token, jwt_required, get_jwt_identity, get_jwt
from sqlalchemy.exc import IntegrityError

from app.models.blacklist_token import BlacklistToken
from app.models.user import Users
from app.extension import db, jwt
from app.auth import authBp
```
Code ini digunakan untuk melakukan import dari beberapa module agar dapat menggunakan fungsi yang ada pada module tersebut.

```
@authBp.route('/register', methods=['POST'], strict_slashes = False)
def register():
    data = request.get_json()

    name = data.get('name', None)
    email = data.get('email', None)
    password = generate_password_hash(data.get('password', None))

    if not name or not email or not password:
        return jsonify({
            "message": "Name or Email or Password is required!"
        }), 400
    
    try:
        db.session.add(Users(name=name, email=email, password=password))
        db.session.commit()
    except IntegrityError:
        return jsonify({
            "message": f"User {name} already registered"
        }), 422
    
    return jsonify({
        "message": "User registration is completed!"
    }), 200
```
Kode yang diberikan adalah penggunaan Blueprint dalam Flask untuk menangani rute '/register/' dan method 'POST'. Fungsi register() dihubungkan ke rute '/register' pada blueprint 'authBp'. Dalam fungsi register(), pertama-tama akan mengambil data JSON menggunakan 'register.get_json()', data yang diambil data 'name', 'email', dan 'password'.

Setelah mendapatkan data, akan dilakukan pemeriksaan terhadap ketersediaan data yang ada. Jika ada data yang kosong, makan akan dikembalikan pesan error dan status kode 400.

Jika data sudah terisi semua, makan data pengguna baru akan dimasukkan kedalam database. Jika terdapat kesalahan 'IntegrityError', itu menyatakan bahwa nama user sudah tersedia, dan akan dikembalikan pesan bahwa nama sudah terdaftar dan status kode 422

Jika tidak ada kesalahan, pengguna baru berhasil ditambahkan kedalam database. Sebuah respon akan dikembalikan dan status kode 200

```
@authBp.route('/login', methods=['POST'], strict_slashes=False)
def login():
    data = request.get_json()

    email = data.get('email', None)
    password = data.get('password', None)

    if not email or not password:
        return jsonify({
            "message": "Email or Password is required!"
        }), 400
    
    user = Users.query.filter_by(email=email).first()

    if not user or not check_password_hash(user.password, password):
        return jsonify({
            "message": "Email or Password is invalid!"
        }), 422
    
    access_token = create_access_token(identity=user.id)
    refresh_token = create_refresh_token(identity=user.id)

    return jsonify({
        "message": "Login success!",
        "access_token": access_token,
        "refresh_token": refresh_token
    }), 200
```
Kode yang diberikan adalah penggunaan Blueprint dalam Flask untuk menangani rute '/login/' dan method 'POST'. Fungsi login() dihubungkan ke rute '/login' pada blueprint 'authBp'. Dalam fungsi login(), pertama-tama akan mengambil data JSON menggunakan 'register.get_json()', data yang diambil data 'email', dan 'password'.

Setelah mendapatkan data, akan dilakukan pemeriksaan terhadap ketersediaan data yang ada. Jika ada data yang kosong, makan akan dikembalikan pesan error dan status kode 400.

Setelah memastikan data sudah diisi, maka selanjutnya akan dilakukan pemeriksaan di database apakah email dan password sudah sesuai. Jika pengguna tidak ditemukan, maka akan dikembalikan respon berupa pesan dan status kode 422
Jika tidak ada kesalahan, maka akses token akan dibuat dengan refresh token menggunakan fungsi 'create_access_token()' dan 'create_refresh_token()'

```
@authBp.route('/refresh', methods=['POST'])
@jwt_required(refresh=True)
def refresh_token():
    user = get_jwt_identity()
    access_token = {
        "access_token": create_access_token(identity=user)
    }

    return jsonify(access_token), 200

@authBp.route('/logout', methods=['POST'], strict_slashes = False)
@jwt_required(locations=['headers'])
def logout():
    raw_jwt = get_jwt()

    jti = raw_jwt.get('jti')
    token = BlacklistToken(jti = jti)

    db.session.add(token)
    db.session.commit()

    return jsonify({"message": "Logout success!"}), 200

@jwt.token_in_blocklist_loader
def check_if_token_is_revoked(jwt_header, jwt_payload: dict):
    jti = jwt_payload["jti"]
    token = BlacklistToken.query.filter_by(jti=jti).first()

    return token is not None
```
Fungsi 'refresh_token()' dihubungkan ke route /refresh pada blueprint authBp dengan method 'POST'. Didalam fungsi terdapat @jwt_required(refresh=True) untuk memastikan hanya pengguna yang memiliki refresh token yang valid dapat mengaksesnya. Di dalam fungsi, kita mendapatkan identitas pengguna dari refresh token yang valid menggunakan 'get_jwt_identity()'. Kemudian, kita membuat akses token baru menggunakan 'create_access_token()' dengan identitas pengguna yang diperoleh. Akses token baru tersebut dikembalikan dalam respons JSON dengan status kode 200. Tujuan dari fungsi ini adalah untuk memberikan akses token baru kepada pengguna setelah refresh token mereka divalidasi.

Fungsi 'logout()' dihubungkan ke route '/logout' pada blueprint authBp dengan method 'POST'. Didalam fungsi terdapat decorator @jwt_required(locations=['headers']) untuk memastikan hanya pengguna yang mengirimkan token akses dalam header permintaan yang dapat melakukan logout. Di dalam fungsi, kita mendapatkan informasi token yang digunakan dalam permintaan menggunakan get_jwt(). Selanjutnya, kita mengambil "jti" (JWT ID) dari token tersebut dan membuat objek BlacklistToken dengan menggunakan nilai "jti". Objek BlacklistToken kemudian ditambahkan ke database untuk memasukkan token ke dalam daftar token yang diambil. Setelah itu, perubahan pada basis data dikonfirmasi dengan melakukan db.session.commit(). Terakhir, sebuah respons JSON dikembalikan dengan pesan bahwa logout berhasil dan status kode 200. Tujuan dari fungsi ini adalah untuk mencabut token akses yang digunakan sehingga token tersebut tidak lagi valid dan tidak dapat digunakan untuk mengakses sumber daya terproteksi.

Fungsi 'check_if_token_is_revoked()' adalah fungsi yang digunakan untuk menentukan apakah suatu token akses ditarik atau dicabut. Fungsi ini ditentukan dengan '@jwt.token_in_blocklist_loader' untuk JWT manager yang digunakan dalam aplikasi. Di dalam fungsi, kita mengambil "jti" (JWT ID) dari JWT payload dan mencari token dengan "jti" tersebut dalam database menggunakan BlacklistToken. Jika token ditemukan, berarti token tersebut telah ditarik dan fungsi ini mengembalikan nilai True. Jika token tidak ditemukan, berarti token masih valid dan fungsi ini mengembalikan nilai False. Dengan menggunakan fungsi ini, kita dapat memastikan bahwa token akses yang ditarik atau dicabut akan dianggap tidak valid saat melakukan verifikasi otentikasi.

### Code 3
Code terdapat di app/frontend/__init__.py
```
from flask import Blueprint
from flask_cors import CORS

frontendBp = Blueprint('frontend', __name__)
CORS(frontendBp)
from app.frontend import routes
```
Code ini terdari import Blueprint dan CORS agar dapat menggunakan fungsi yang disediakan dari Flask dan Flask-CORS. Kemudian 'frontendBp' digunakan sebagai objek Blueprint agar bisa menggunakan fungsi Blueprint yang disediakan Flask, Blueprint diberi nama 'frontend'. Setelah itu CORS diterapkan menggunakan objek 'frontendBp' yang telah dibuat sebelumnya. 

### Code 4
app/frontend/routes.py
```
from app.frontend import frontendBp
from flask import render_template
from flask_jwt_extended import jwt_required, get_jwt_identity

@frontendBp.route('', strict_slashes = False)
def home():
    return render_template('tasks/todo.html')

@frontendBp.route('auth/login', strict_slashes = False)
def login():
    return render_template('/auth/login.html')

@frontendBp.route('auth/register', strict_slashes = False)
def register():
    return render_template('/auth/register.html')
```
Pertama-tama akan dilakukan import dari module untuk menggunakan fungsi yang ada pada module tersebut.

Fungsi 'home()' digunakan sebagai rute utama pada blueprint frontendBp. Ketika permintaan diterima, maka fungsi 'home()' akan dijalankan, sehingga fungsi akan mengembalikan tampilan html yang berada di 'tasks/todo.html' dengan menggunakan 'render_template()'

Fungsi 'login()' digunakan sebagai rute untuk halaman login pada blueprint frontendBp yang berada di 'auth/login'. Ketika permintaan diterima, maka fungsi 'login()' akan dijalankan, sehingga fungsi akan mengembalikan tampilan html yang berada di '/auth/login.html' dengan menggunakan 'render_template()'

Fungsi register() digunakan sebagai rute untuk halaman registrasi pada blueprint frontendBp yang berada di auth/register Ketika permintaan diterima untuk rute ini, fungsi 'register()' akan dijalankan. Fungsi ini mengembalikan tampilan HTML yang dibuat menggunakan template '/auth/register.html' dengan menggunakan 'render_template()'.

### Code 5
Code ini berada di app/models/blacklist_token.py
```
from app.extension import db

class BlacklistToken(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    jti = db.Column(db.String(36), nullable=False, unique=True)

    def serialize(self):
        return {
            "id": self.id,
            "jti": self.jti
        }
```
Pertama module 'db' diimport dari 'app.extension'. Kelas 'BlacklistToken' didefinisikan sebagai subclass dari 'db.Model' yang merupakan kelas dasar untuk menggunakan SQLAlchemy dalam Flask. method 'serialize(self)' untuk mengembalikan representasi serialisasi objek dalam bentuk dictionary. Dictionary berisi pasangan kunci yang mewakili atribut objek, yaitu 'id' dan 'jti'. Hal ini dilakukan untuk mengubah ke format yang lebih mudah dikirim.

### Code 6
Code ini berada di app/models/task.py
```
from app.extension import db

class Tasks(db.Model):
    id = db.Column(db.Integer, primary_key = True)
    title = db.Column(db.String(64), nullable=False)
    description = db.Column(db.String(1024))
    status = db.Column(db.Boolean, default = False)
    user_id = db.Column(db.Integer, db.ForeignKey('users.id'))
    user = db.relationship('Users', back_populates = 'tasks')

    def serialize(self): 
        return {
            "id": self.id,
            "title": self.title,
            "description":self.description,
            "status": self.status,
            "user_id":self.user_id
        }

```
Pertama module 'db' diimport dari 'app.extension'. Kelas 'Tasks' didefinisikan sebagai subclass dari 'db.Model' yang merupakan kelas dasar untuk menggunakan SQLAlchemy dalam Flask. Kelas ini memiliki beberapa kolom dalam tabel database, yaitu 'id', 'title', 'description', 'status', dan 'user_id'. 'user = db.relationship('Users', back_populates = 'tasks')' digunakan untuk mendefinikan relasi antara tabel 'tasks' dan 'users', yang berarti 'tasks' terhubung dengan satu pengguna 'users'.

Method 'serialize(self)' digunakan untuk mengembalikan representasi serialisasi objek dalam bentuk dictionary. Dictionary berisi pasangan kunci yang mewakili atribut objek, yaitu 'id', 'title', 'description', 'status', dan 'user_id'. Hal ini dilakukan untuk mengubah ke format yang lebih mudah dikirim.

### Code 7
Code ini terdapat pada app/models/user.py
```
from app.extension import db

# table database user
class Users(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), unique=True, nullable=False)
    email = db.Column(db.String(128), nullable=False)
    password = db.Column(db.String(1024), nullable=False)
    tasks = db.relationship('Tasks', back_populates='user')

    # fungsi serialize untuk mengembalikan data dictionary
    def serialize(self): 
        return {
            "id": self.id,
            "name": self.name,
            "email":self.email
        }
```
Pertama module 'db' diimport dari 'app.extension'. Kelas 'Users' didefinisikan sebagai subclass dari 'db.Model' yang merupakan kelas dasar untuk menggunakan SQLAlchemy dalam Flask. Kelas ini memiliki beberapa kolom dalam tabel database, yaitu 'id', 'name', 'email', dan 'password'. 'tasks = db.relationship('Tasks', back_populates = 'user')' digunakan untuk mendefinikan relasi antara tabel 'tasks' dan 'users', yang berarti 'users' terhubung 'tasks'.

Method 'serialize(self)' digunakan untuk mengembalikan representasi serialisasi objek dalam bentuk dictionary. Dictionary berisi pasangan kunci yang mewakili atribut objek, yaitu 'id', 'name', dan 'email'. Hal ini dilakukan untuk mengubah ke format yang lebih mudah dikirim.

### Code 8
Code ini berada di app/static/css/login.css
```
.glass-background{
background: rgba(0, 0, 0, 0.29);
border-radius: 16px;
box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
backdrop-filter: blur(3.3px);
-webkit-backdrop-filter: blur(3.3px);
border: 1px solid rgba(0, 0, 0, 0.4);
}

.bg-images{
    background-image: url("../images/black-bg2.png"); 
    background-position: center;
    background-repeat: no-repeat;
    background-size: cover;
    height: 100%;
}

.btn-circle-md {
    width: 80px;
    height: 80px;
    border-radius: 50px;
    font-size: 2.5rem;
    text-align: center;
}
```
Code ini digunakan untuk memodifikasi elemen-elemen yang digunakan pada .html.

### Code 9
Code ini berada di app/static/js/script-login.js
```
const formLogin = document.getElementById("form-login")
const API_HOST = 'http://127.0.0.1:5000/api'
```
'const formLogin' dan 'const API_HOST' digunakan untuk mendeklarasikan variabel formLogin yang mereferensikan elemen dengan id "form-login". Kemudian, konstanta API_HOST diatur dengan URL host API yang akan digunakan untuk mengirim permintaan ke server.
```
window.onload = function() {
```
Fungsi dijalankan ketika halaman telah selesai dimuat. Seluruh kode yang ada di dalam fungsi ini akan dieksekusi setelah elemen halaman selesai dimuat.
```
formLogin.addEventListener('submit', (e) =>{
e.preventDefault();
```
Event listener yang akan merespons saat formulir login di-submit. Saat formulir di-submit, event listener ini mencegah perilaku default (refresh halaman) menggunakan e.preventDefault().
```
const xhr = new XMLHttpRequest()
const url = API_HOST + "/auth/login"
```
Variabel 'xhr' dideklarasikan sebagai objek 'XMLHttpRequest' yang digunakan untuk membuat permintaan HTTP ke server. Variabel 'url' diatur dengan URL lengkap untuk rute login pada API.
```
const email = document.getElementById("email").value
const password = document.getElementById("password").value
```
Nilai email dan password diambil dari elemen dengan id "email" dan "password" menggunakan 'getElementById().value'.
```
const toastTrigger = document.getElementById("liveToast");
const toastMsg = document.getElementById("toast-body");
toastMsg.innerHTML = "email atau password salah"
const toast = new bootstrap.Toast(toastTrigger)

if (email == '' || password == ''){
    return toast.show()
        }
```
Ini adalah bagian yang menangani validasi dan menampilkan pesan kesalahan jika email atau password kosong. Jika email atau password kosong, maka sebuah toast (pemberitahuan sementara) akan ditampilkan menggunakan Bootstrap.
```
const data = JSON.stringify({
    email: email,
    password: password,
})
```
Objek data dihasilkan dengan mengkonversi email dan password ke dalam format JSON menggunakan JSON.stringify().
```
xhr.open("POST", url, true)
xhr.setRequestHeader('Content-Type', 'application/json;charset=utf-8')
xhr.onreadystatechange = function(){
...
xhr.send(data)}
```
Ini adalah bagian yang menyiapkan dan mengirim permintaan POST menggunakan objek 'xhr'. 'xhr.open()' digunakan untuk mengkonfigurasi permintaan dengan metode 'POST' dan URL yang telah ditentukan sebelumnya. Kemudian, 'xhr.setRequestHeader()' digunakan untuk mengatur header permintaan dengan tipe konten yang sesuai. 'xhr.onreadystatechange' adalah fungsi yang akan dipanggil saat status permintaan berubah. Terakhir, 'xhr.send()' mengirimkan data dalam format JSON ke server.
```
if(this.status == 200){
    const response = JSON.parse(this.response)
    localStorage.setItem('access_token', response.access_token)
    window.location.href = "http://127.0.0.1:5000/"
} else {
    toastMsg.innerHTML = this.response
    toast.show()
}
```
Ini adalah bagian yang menangani respons dari server setelah permintaan login dikirim. Jika status permintaan adalah 200 (OK), respons JSON dari server akan diuraikan menggunakan 'JSON.parse()'. Akses token yang diterima dari respons akan disimpan dalam penyimpanan lokal (localStorage) dengan nama 'access_token'. Selain itu, jika status permintaan bukan 200, pesan kesalahan dari server akan ditampilkan pada toast dan ditampilkan kepada pengguna.

### Code 10
Code ini berada di app/static/js/script-register.js
```
const formRegistration = document.getElementById("form-registration")
const API_HOST = 'http://127.0.0.1:5000/api'
```
Kode ini mendeklarasikan variabel 'formRegistration' yang mereferensikan elemen dengan id "form-registration". Kemudian, konstanta 'API_HOST' diatur dengan URL host API yang akan digunakan untuk mengirim permintaan ke server.
```
formRegistration.addEventListener('submit', (e) =>{
    e.preventDefault();
```
Ini adalah event listener yang akan merespons saat formulir registrasi di-submit. Saat formulir di-submit, event listener ini mencegah perilaku default (refresh halaman) menggunakan 'e.preventDefault()'.
```
const xhr = new XMLHttpRequest();
const url = API_HOST + "/auth/register"
```
Variabel 'xhr' dideklarasikan sebagai objek 'XMLHttpRequest' yang digunakan untuk membuat permintaan HTTP ke server. Variabel 'url' diatur dengan URL lengkap untuk rute registrasi pada API.
```
const name = document.getElementById("nama-lengkap").value
const email = document.getElementById("email").value
const password = document.getElementById("password").value
const confirmPassword = document.getElementById("confirm-password").value
```
Nilai name, email, password, dan confirmPassword diambil dari elemen dengan id yang sesuai menggunakan 'getElementById().value'.
```
const toastTrigger = document.getElementById("liveToast");
const toastMsg = document.getElementById("toast-body");
```
Variabel toastTrigger dan toastMsg mereferensikan elemen yang akan digunakan untuk menampilkan pesan kesalahan atau konfirmasi menggunakan toast.
```
if(!name || !email || !password || !confirmPassword){
    toastMsg.innerHTML = "Form harus diisi semua"
    const toast = new bootstrap.Toast(toastTrigger)
    return toast.show()
}
if(password != confirmPassword){
    toastMsg.innerHTML = "password yang dimasukkan tidak cocok"
    return toast.show()
}
```
Yang pertama adalah bagian yang menangani validasi form registrasi. Jika salah satu field tidak diisi, maka pesan kesalahan akan ditampilkan pada toast dan halaman tidak akan dikirim ke server. Yang kedua adalah bagian yang menangani validasi konfirmasi password. Jika password dan konfirmasi password tidak cocok, maka pesan kesalahan akan ditampilkan pada toast dan halaman tidak akan dikirim ke server.
```
const data = JSON.stringify({
    name: name,
    email: email,
    password: password,
})
```
Objek data dihasilkan dengan mengkonversi name, email, dan password ke dalam format JSON menggunakan 'JSON.stringify()'.
```
xhr.open('POST', url, true)
xhr.setRequestHeader('Content-Type', 'application/json;charset=utf-8')
xhr.onreadystatechange = function(){
...
xhr.send(data)}
```
Ini adalah bagian yang menyiapkan dan mengirim permintaan 'POST' menggunakan objek 'xhr'. 'xhr.open()' digunakan untuk mengkonfigurasi permintaan dengan metode 'POST' dan URL yang telah ditentukan sebelumnya. Kemudian, 'xhr.setRequestHeader()' digunakan untuk mengatur header permintaan dengan tipe konten yang sesuai. 'xhr.onreadystatechange' adalah fungsi yang akan dipanggil saat status permintaan berubah. Terakhir, 'xhr.send()' mengirimkan data dalam format JSON ke server.
```
if(this.status == 200){
    toastMsg.innerHTML = "User Registration Success"
    const toast = new bootstrap.Toast(toastTrigger)
    toast.show()
    window.location.href = "http://127.0.0.1:5000/auth/login"
} else{
    toastMsg.innerHTML = this.response
    toast.show()
}
```
Ini adalah bagian yang menangani respons dari server setelah permintaan registrasi dikirim. Jika status permintaan adalah 200 (OK), maka pesan konfirmasi akan ditampilkan pada toast dan pengguna akan diarahkan ke halaman login. Jika status permintaan bukan 200, pesan kesalahan dari server akan ditampilkan pada toast dan ditampilkan kepada pengguna.

### Code 11
Code ini berada di app/static/js/script-todo.js
```
const API_HOST = "http://127.0.0.1:5000/api"
```
Variabel 'API_HOST' diatur dengan URL host API yang akan digunakan untuk mengirim permintaan ke server.
```
function dragStart(event){
    event.dataTransfer.setData("todo", event.target.id)
}
```
Fungsi 'dragStart()' dijalankan saat operasi seret dimulai pada elemen target. Ini mengatur data yang akan dikirimkan selama operasi seret menggunakan metode 'setData()' pada objek 'dataTransfer' pada event.
```
function drop(event){
    event.preventDefault();
    const data = event.dataTransfer.getData("todo")
    event.target.appendChild(document.getElementById(data))
    const dataId = event.srcElement.lastChild.id
    checkStatus(dataId)
}
```
Fungsi 'drop()' dijalankan saat elemen yang sedang dijatuhkan di atas elemen target. Ini mencegah perilaku default yang biasanya terjadi saat menjatuhkan elemen. Kemudian, fungsi ini mengambil data yang diset pada operasi seret menggunakan metode 'getData()' pada objek 'dataTransfer'. Selanjutnya, elemen yang dijatuhkan ditambahkan sebagai anak pada elemen target menggunakan metode 'appendChild()'. Terakhir, fungsi 'checkStatus()' dipanggil dengan menggunakan ID elemen terakhir yang ditambahkan untuk memeriksa dan memperbarui status tugas terkait.
```
function allowDrop(event){
    event.preventDefault();
}
```
Fungsi allowDrop() mencegah perilaku default yang biasanya terjadi saat menjatuhkan elemen pada area target. Ini mencegah browser melakukan tindakan default yang tidak diinginkan.
```
function updateStatus(id, status){
    const xhr = new XMLHttpRequest()
    const url = API_HOST + '/tasks/status' + id
    const data = JSON.stringify({
        status: !status
    })
```
Fungsi ini menerima dua parameter: 'id' dan 'status'. Variabel 'xhr' dideklarasikan sebagai objek 'XMLHttpRequest' yang akan digunakan untuk membuat permintaan HTTP. url ditentukan dengan menggabungkan 'API_HOST' dan jalur '/tasks/status' dengan id tugas yang diberikan. Variabel 'data' diatur dengan objek JSON yang berisi 'status' yang dibalik dengan operator '!'.
```
xhr.open('PUT', url, true)
xhr.setRequestHeader('Content-Type', 'application/json;charset=utf-8')
xhr.setRequestHeader("Authorization", `Bearer ${localStorage.getItem('access_token')}`)
```
Pada bagian ini, metode 'open()' dipanggil pada objek 'xhr' untuk mengatur permintaan dengan metode PUT dan URL yang telah ditentukan. Kemudian, 'setRequestHeader()' digunakan untuk mengatur header permintaan. Header 'Content-Type' diatur sebagai 'application/json;charset=utf-8' untuk menunjukkan bahwa data dikirim dalam format JSON. Selain itu, header 'Authorization' diatur dengan token akses yang diperoleh dari penyimpanan lokal menggunakan localStorage.getItem('access_token'). Ini digunakan untuk mengotentikasi permintaan ke API.
```
xhr.onreadystatechange = function(){
        if(this.readyState == 4 && this.status == 200){
            location.reload()
        }
    }
    xhr.send(data)
}
```
Fungsi ini mengatur properti 'onreadystatechange' pada objek 'xhr' untuk menentukan fungsi yang akan dipanggil saat status permintaan berubah. Dalam hal ini, jika 'readyState' adalah 4 (permintaan selesai) dan 'status' adalah 200 (OK), maka 'location.reload()' dipanggil untuk me-refresh ulang halaman saat tugas berhasil diperbarui. Terakhir, permintaan dikirim dengan menggunakan metode 'send()' pada objek 'xhr' dan data yang dikirimkan adalah objek JSON yang telah di-stringify dalam variabel 'data'
```
function checkStatus(id){
    const xhr = new XMLHttpRequest
    const url = API_HOST + '/tasks/' + id

    xhr.open('GET', url, true)
    xhr.setRequestHeader('Authorization', `Bearer ${localStorage.getItem('access_token')}`)
```
Fungsi ini menerima satu parameter yaitu id. Variabel 'xhr' dideklarasikan sebagai objek 'XMLHttpRequest' yang akan digunakan untuk membuat permintaan HTTP. 'url' ditentukan dengan menggabungkan API_HOST dengan jalur '/tasks/' dan 'id' tugas yang diberikan. Metode 'open()' dipanggil pada objek 'xhr' untuk mengatur permintaan dengan metode GET dan URL yang telah ditentukan. Header 'Authorization' diatur dengan token akses yang diperoleh dari penyimpanan lokal menggunakan 'localStorage.getItem('access_token')'. Ini digunakan untuk mengotentikasi permintaan ke API.
```
xhr.onreadystatechange = function(){
    if(this.readyState == 4 && this.status == 200){
    const response = JSON.parse(this.response)

    updateStatus(id, response.data.status)
    }
}
return xhr.send()
}
```
Fungsi ini mengatur properti 'onreadystatechange' pada objek 'xhr' untuk menentukan fungsi yang akan dipanggil saat status permintaan berubah. Dalam hal ini, jika 'readyState' adalah 4 (permintaan selesai) dan 'status' adalah 200 (OK), maka JSON.parse() digunakan untuk mengurai data respons menjadi objek JavaScript. Objek 'response' mengandung data tugas yang diperoleh dari API, termasuk status tugas. Kemudian, fungsi 'updateStatus()' dipanggil dengan menggunakan 'id' tugas dan status yang diperoleh dari respons API.
```
const todoItem = document.getElementById("todo-item")
const doneItem = document.getElementById("done")
```
Variabel 'todoItem' dan 'doneItem' diatur untuk menyimpan referensi ke elemen HTML di mana tugas akan ditampilkan. 'todoItem' merepresentasikan daftar tugas yang belum selesai, sedangkan 'doneItem' merepresentasikan daftar tugas yang sudah selesai.
```
window.onload = function(e){
    const token = localStorage.getItem('access_token')
    if(!token){
        window.location.href = "http://127.0.0.1:5000/auth/login"
    }
```
Event 'window.onload' digunakan untuk menjalankan kode saat halaman web selesai dimuat. Pada bagian ini, kode memeriksa apakah ada token akses yang disimpan di penyimpanan lokal. Jika token tidak ada, pengguna akan dialihkan ke halaman login dengan mengubah 'window.location.href' menjadi URL halaman login yang sesuai.
```
const xhr = new XMLHttpRequest();
const url = API_HOST +"/tasks"

xhr.open("GET", url, true)
xhr.setRequestHeader("Authorization", `Bearer ${token}`)
xhr.onreadystatechange = function(){
    if(this.readyState == 4 & this.status == 200){
        const tasks = JSON.parse(this.response)
        tasks['data'].forEach((task) => {
            const article = document.createElement("article")
            const badgeDelete = document.createElement('button')
            const badgeEdit = document.createElement('button')
            const badgeDone = document.createElement('button')
            const h4 = document.createElement('h4')
            const p = document.createElement('p')

            h4.appendChild(document.createTextNode(task.title))
            h4.setAttribute('id', task.id)
            p.appendChild(document.createTextNode(task.description))

            article.setAttribute('class', 'border p-3 drag')
            article.setAttribute('ondragstart', "drag(event)")
            article.setAttribute("draggable", "true")
            article.setAttribute("id", task.id)

            badgeDelete.setAttribute('class', 'badge bg-danger')
            badgeDelete.setAttribute("href", "#")
            badgeDelete.setAttribute("data-id", task.id)
            badgeDelete.setAttribute("data-bs-toggle", "modal")
            badgeDelete.setAttribute("data-bs-target", "#modalDelete")
                
            badgeDelete.appendChild(document.createTextNode("Delete"))

            badgeEdit.setAttribute('class', 'badge bg-info')
            badgeEdit.setAttribute("href", "#")
            badgeEdit.setAttribute("data-title", task.title)
            badgeEdit.setAttribute('data-description', task.description)
            badgeEdit.setAttribute("data-id", task.id)
            badgeEdit.setAttribute("data-bs-toggle", "modal")
            badgeEdit.setAttribute("data-bs-target", "#modalEdit")
            
            badgeEdit.appendChild(document.createTextNode("Edit"))

            badgeDone.setAttribute('class', 'badge bg-success')
            badgeDone.setAttribute("href", "#")
            badgeDone.setAttribute("data-title", task.title)
            badgeDone.setAttribute('data-description', task.description)
            badgeDone.setAttribute("data-id", task.id)
            badgeDone.setAttribute("data-bs-toggle", "modal")
            badgeDone.setAttribute("data-bs-target", "#modalDone")
            
            badgeDone.appendChild(document.createTextNode("Done"))

            article.appendChild(h4)
            article.appendChild(p)
            article.appendChild(badgeDelete)
            article.appendChild(badgeEdit)
            article.appendChild(badgeDone)

            if(task.status == true){
                article.setAttribute('style', 'text-decoration:line-through')
                doneItem.appendChild(article)
            } else{
                todoItem.appendChild(article)
                }
            })
        }
    }
    xhr.send()
}
```
Pada bagian ini, 'XMLHttpRequest' digunakan untuk mengirim permintaan GET ke API untuk mendapatkan daftar tugas. URL permintaan diatur dengan menggabungkan 'API_HOST' dengan jalur '/tasks'. Header permintaan diatur dengan menyertakan token akses dalam header "Authorization" menggunakan 'setRequestHeader()'. Setelah permintaan dikirim, fungsi 'onreadystatechange' akan dipanggil saat status permintaan berubah. Jika status permintaan adalah 4 (permintaan selesai) dan status HTTP adalah 200 (OK), kode akan melanjutkan untuk memproses respons API.

Respons dari API berupa daftar tugas dalam bentuk objek JSON. Melalui penggunaan 'JSON.parse()', data tugas diurai menjadi objek JavaScript. Kemudian, setiap tugas dalam daftar '(tasks['data'])' diproses menggunakan metode 'forEach()'. Untuk setiap tugas, elemen HTML yang sesuai dibuat dan diatur dengan konten dan atribut yang sesuai. Elemen tugas kemudian ditambahkan ke dalam elemen yang sesuai ('todoItem' atau 'doneItem') berdasarkan status tugas.
```
const addForm = document.getElementById("form-add");
addForm.addEventListener("submit", function (event) {
  event.preventDefault();
```
Kode ini menangkap acara submit pada formulir dengan id "form-add" dan mencegah perilaku default form submit agar halaman tidak refresh saat tombol submit ditekan.
```
let xhr = new XMLHttpRequest();
let url = API_HOST + "/tasks";
let title = document.getElementById("title").value;
let description = document.getElementById("description").value;
```
Variabel 'xhr' dideklarasikan sebagai objek 'XMLHttpRequest' yang akan digunakan untuk membuat permintaan HTTP. Variabel 'url' diatur untuk menentukan URL endpoint API untuk menambahkan tugas. Nilai input dari elemen dengan id "title" dan "description" dipilih dan disimpan dalam variabel 'title' dan 'description'.
```
const toastLiveExample = document.getElementById("liveToastAdd");
const toastMsgAdd = document.getElementById("toast-body-add");
const toast = new bootstrap.Toast(toastLiveExample);
```
Variabel 'toastLiveExample' dan 'toastMsgAdd' digunakan untuk mengambil elemen yang berkaitan dengan tampilan toast. Variabel 'toast' akan menjadi objek toast yang akan digunakan untuk menampilkan pesan.
```
if (title == "") {
    toastMsgAdd.innerHTML = "Isian title tidak boleh kosong";
    toast.show();
}
if (description == "") {
    toastMsgAdd.innerHTML = "Isian deskripsi tidak boleh kosong";
    toast.show();
}
```
Kode ini melakukan validasi input untuk memastikan bahwa input "title" dan "description" tidak kosong. Jika salah satu atau kedua input kosong, maka pesan kesalahan akan ditampilkan menggunakan toast.
```
let new_data = JSON.stringify({
    title: title,
    description: description,
});

xhr.open("POST", url, true);
xhr.setRequestHeader("Content-Type", "application/json;charset=utf-8");
xhr.setRequestHeader("Authorization",`Bearer ${localStorage.getItem("access_token")}`);
```
Objek 'new_data' dibuat untuk menyimpan data tugas yang akan dikirim ke API. Objek ini diinisialisasi dengan menggunakan 'JSON.stringify()' untuk mengubah data menjadi format JSON. Selanjutnya, metode 'open()' pada objek 'xhr' dipanggil untuk mengatur permintaan dengan metode POST dan URL yang telah ditentukan. Header permintaan "Content-Type" diatur sebagai "application/json" untuk menunjukkan bahwa data dikirim dalam format JSON. Header "Authorization" diatur dengan menyertakan token akses yang diperoleh dari penyimpanan lokal menggunakan 'localStorage.getItem("access_token")'. Ini digunakan untuk mengotentikasi permintaan ke API.
```
xhr.onreadystatechange = function () {
    if (this.readyState == 4 && this.status == 200) {
        const myModalAdd = bootstrap.Modal.getInstance("#modalAdd");
        myModalAdd.hide();

        addForm.reset();
        location.reload();
    } else {
        const toastLive = document.getElementById("liveToastAdd");
        const toastMsg = document.getElementById("toast-body-add");
        const toast = new bootstrap.Toast(toastLive);
        toastMsg.innerHTML = "Data berhasil";
        toast.show();
    }
  };
  xhr.send(new_data);
});
```
Kode ini menetapkan fungsi 'onreadystatechange' pada objek 'xhr' untuk menentukan tindakan yang akan diambil saat status permintaan berubah. Jika status permintaan adalah 4 (permintaan selesai) dan status HTTP adalah 200 (OK), maka tindakan berikut akan dijalankan:

1. Menutup modal dengan id "modalAdd" menggunakan 'Modal.getInstance("#modalAdd").hide()'.
2. Mengatur ulang formulir dengan 'addForm.reset()' untuk menghapus nilai input yang telah dimasukkan.
3. Memuat ulang halaman dengan 'location.reload()' untuk memperbarui tampilan daftar tugas.

Jika status permintaan tidak berhasil (status tidak sama dengan 200), maka pesan berhasil akan ditampilkan menggunakan toast.
```
const myModalEdit = document.getElementById("modalEdit");
myModalEdit.addEventListener("show.bs.modal", function (event) {
  let dataId = event.relatedTarget.attributes["data-id"];
```
Kode tersebut mengatur event handler untuk modal edit (myModalEdit) ketika event show.bs.modal dan mengambil atribut data-id dari elemen terkait yang memicu event (relatedTarget). Atribut ini menyimpan ID dari data yang akan diedit
```
let xhr = new XMLHttpRequest();
let url = API_HOST + "/tasks/" + dataId.value;

xhr.open("GET", url, true);
xhr.setRequestHeader("Content-Type", "application/json;charset=utf-8");
xhr.setRequestHeader("Authorization",`Bearer ${localStorage.getItem("access_token")}`
);
xhr.onreadystatechange = function () {
    if (this.readyState == 4 && this.status == 200) {
        let data = JSON.parse(this.response);
        let oldTitle = document.getElementById("edit-title");
        let oldDescription = document.getElementById("edit-description");
        oldTitle.value = data.data.title;
        oldDescription.value = data.data.description;
    }
};
  xhr.send();
```
Kode ini membuat permintaan GET menggunakan objek XMLHttpRequest untuk mengambil data dengan ID yang spesifik. Permintaan dikirim ke URL yang sesuai dengan ID data yang diambil sebelumnya. Permintaan ini juga memasukkan header yang diperlukan, seperti "Content-Type" dan "Authorization". Ketika tanggapan berhasil diterima (status 200), data yang diterima diuraikan dari format JSON menjadi objek JavaScript. Nilai judul dan deskripsi kemudian ditetapkan pada elemen dengan ID "edit-title" dan "edit-description" untuk menampilkan data yang saat ini ada dalam formulir modal.
```
let editForm = document.getElementById("form-edit");
editForm.addEventListener("submit", function (event) {
    event.preventDefault();
    let xhr = new XMLHttpRequest();
    let url = API_HOST + "/tasks/" + dataId.value;

    let newTitle = document.getElementById("edit-title").value;
    let newDescription = document.getElementById("edit-description").value;
    const toastLiveExample = document.getElementById("liveToastEdit");
    const toastMsgEdit = document.getElementById("toast-body-edit");
    const toast = new bootstrap.Toast(toastLiveExample);
    if (newTitle == "") {
        toastMsgEdit.innerHTML = "isian title tidak boleh kosong";
        toast.show();
    }
    if (newDescription == "") {
        toastMsgEdit.innerHTML = "isian description tidak boleh kosong";
        toast.show();
    }
```
Kode ini menangani event pengiriman formulir edit. Ketika formulir dikirim, fungsi yang ditentukan akan dieksekusi. Panggilan 'event.preventDefault()' mencegah perilaku bawaan dari formulir yang akan memperbarui halaman. Selanjutnya, nilai judul dan deskripsi yang diinput oleh pengguna diambil dari elemen formulir. Sebuah toast didefinisikan menggunakan Bootstrap Toast untuk menampilkan pesan yang relevan. Kemudian, validasi dilakukan untuk memastikan bahwa kedua input tidak kosong. Jika validasi tidak berhasil, pesan kesalahan akan ditampilkan pada toast yang sesuai.
```
```
const myModalDelete = document.getElementById("modalDelete");
myModalDelete.addEventListener("show.bs.modal", function (event) {
  let dataId = event.relatedTarget.attributes["data-id"];
```
Kode ini mengambil referensi elemen modal delete menggunakan ID "modalDelete" dari elemen HTML. Kode ini menambahkan event listener ke modal delete ketika event show.bs.modal terjadi. Saat modal delete ditampilkan, fungsi yang ditentukan akan dieksekusi. Kode ini mendapatkan ID data yang dihapus dari atribut "data-id" elemen yang diklik oleh pengguna. Atribut "data-id" berisi informasi ID yang digunakan untuk mengidentifikasi data yang akan dihapus.
```
```
const deleteForm = document.getElementById("formDelete");
deleteForm.addEventListener("submit", function (event) {
    event.preventDefault();
    let xhr = new XMLHttpRequest();
    let url = API_HOST + "/tasks/" + dataId.value;

    xhr.open("DELETE", url, true);
    xhr.setRequestHeader("Content-Type", "application/json;charset=utf-8");
    xhr.setRequestHeader(
      "Authorization",
      `Bearer ${localStorage.getItem("access_token")}`
    );
    xhr.onreadystatechange = function () {
      if (this.readyState == 4 && this.status == 200) {
        let response = JSON.parse(this.response);

        const myModalDelete = bootstrap.Modal.getInstance("modalDelete");
        myModalDelete.hide();

        const alertLoc = document.getElementById("alert-loc");
        const alertEl = document.createElement("div");
        alertEl.setAttribute("class", "alert alert-success");
        alertEl.setAttribute("role", "alert");
        alertEl.innerHTML = response.message;

        alertLoc.append(alertEl);

        document.getElementById(dataId.value).classList.add("d-none");
      }
    };
    xhr.send();
  });
});
```
Kode ini membuat objek XMLHttpRequest untuk melakukan permintaan DELETE ke API. URL permintaan dibentuk dengan menggunakan URL API dan ID data yang akan dihapus. Header yang diperlukan, seperti "Content-Type" dan "Authorization", juga ditetapkan. Kode ini menangani respons yang diterima setelah permintaan DELETE berhasil dilakukan. Respons JSON diterjemahkan menjadi objek JavaScript. Modal delete ditutup menggunakan myModalDelete.hide(). Selain itu, sebuah elemen alert dibuat untuk menampilkan pesan sukses. Elemen tersebut diberi atribut kelas dan atribut peran, serta mengandung pesan dari respons. Selanjutnya, elemen dengan ID yang sesuai akan diberi kelas "d-none" untuk menyembunyikannya dari tampilan.
```
function showRealTimeClock(){
    const footerTime = document.getElementById("footer-time")
    const time = new Date()
    footerTime.innerHTML = time.toLocaleTimeString([],{
        hour12: false
    });
}

setInterval(showRealTimeClock, 1000)
```
Kode ini mengambil elemen pada halaman dengan ID "footer-time" dan menyimpannya dalam variabel footerTime, kemudian membuat objek Date baru yang akan merepresentasikan waktu aktual saat ini. Kode ini menggunakan metode toLocaleTimeString() pada objek Date untuk mendapatkan waktu aktual dalam bentuk string dengan format yang sesuai dengan pengaturan lokal. Opsi { hour12: false } digunakan untuk menampilkan waktu dalam format 24 jam. Kemudian, waktu yang telah diformat tersebut ditetapkan sebagai isi dari elemen footerTime menggunakan properti innerHTML. Kode ini menggunakan fungsi setInterval untuk memanggil fungsi showRealTimeClock setiap 1000 milidetik (1 detik). Dengan demikian, waktu yang ditampilkan pada elemen "footer-time" akan diperbarui secara otomatis setiap detik.

### Code 12
Code ini berada di app/tasks/__init__.py
```
from flask import Blueprint
from flask_cors import CORS

taskBp = Blueprint('task', __name__)
CORS(taskBp)

from app.task import routes
```
Kode ini mengimpor kelas Blueprint dari modul flask dan fungsi CORS dari modul flask_cors. Kode ini membuat objek Blueprint dengan nama 'task'. Blueprint digunakan untuk mendefinisikan sekumpulan rute (routes) yang terkait dengan suatu bagian atau fitur dalam aplikasi. Kemudian, Kode ini mengaktifkan fitur CORS untuk Blueprint 'task'. CORS memungkinkan permintaan HTTP dari asal yang berbeda untuk mengakses sumber daya (misalnya API) pada server. Kode ini mengimpor modul routes dari package app.task. Modul routes berisi definisi rute-rute HTTP yang terkait dengan fitur tugas (task) dalam aplikasi.

### Code 13
Code ini berada di app/tasks/routes.py
```
@taskBp.route('/', methods=['GET'], strict_slashes = False)
@jwt_required(locations=["headers"])
```
Kode ini mendefinisikan rute HTTP dengan metode GET dan path '/' pada Blueprint 'task'. Ketika rute ini diakses dengan metode GET, fungsi yang terdekorasi di bawahnya akan dieksekusi. Kode ini mengaplikasikan autentikasi JWT pada rute tersebut. Hanya pengguna yang memiliki JWT yang valid dan terkait dengan otorisasi yang sesuai yang dapat mengakses rute ini. Lokasi JWT diatur sebagai header.
```
def get_all_tasks():
    limit = request.args.get('limit', 10)
    current_user_id = get_jwt_identity()
```
Kode ini mendefinisikan fungsi get_all_tasks() sebagai penangan rute HTTP. Fungsi ini akan dieksekusi ketika rute '/' dengan metode GET diakses. Kode ini mengambil nilai parameter query string 'limit' dari permintaan HTTP. Jika parameter tidak ada, nilai default yang digunakan adalah 10. Kode ini mendapatkan identitas pengguna saat ini yang terkait dengan JWT yang valid. Identitas ini bisa digunakan untuk melakukan verifikasi atau aksi berdasarkan pengguna yang sedang login.
```
if type(limit) is not int:
    return jsonify({
    "message": "Invalid Parameter"
}), 400
```
Kode ini memeriksa apakah tipe data dari parameter 'limit' adalah integer. Jika bukan, maka respons JSON dengan pesan kesalahan dikembalikan dengan kode status 400 (Bad Request).
```
tasks = db.session.execute(db.select(Tasks).limit(limit)).scalars()
```
Kode ini menggunakan SQLAlchemy untuk melakukan query ke database dan mengambil daftar tugas. Fungsi db.session.execute() digunakan untuk menjalankan query, sedangkan db.select(Tasks) adalah ekspresi SQLAlchemy yang menghasilkan SELECT statement untuk tabel 'Tasks'. Metode limit() digunakan untuk membatasi jumlah hasil yang diambil dari database.
```
result = []
for task in tasks:
    result.append(task.serialize())

return jsonify({
    "success": True,
    "data": result
}), 200
```
Kode ini mengiterasi melalui setiap tugas dalam daftar tasks dan mengubah setiap tugas menjadi format serialisasi JSON. Fungsi task.serialize() digunakan untuk mengonversi objek tugas menjadi format JSON. Kode ini mengembalikan respons JSON yang berisi daftar tugas dalam format JSON. Kode status 200 (OK) digunakan untuk menandakan bahwa permintaan berhasil.
```
@taskBp.route('<int:id>', methods=['GET'], strict_slashes = False)
@jwt_required(locations=["headers"])
```
Kode ini mendefinisikan rute HTTP dengan metode GET dan path '/int:id' pada Blueprint 'task'. Path ini mengandung parameter <int:id> yang akan digunakan untuk menyimpan nilai ID dari tugas yang akan diambil. Ketika rute ini diakses dengan metode GET, fungsi yang terdekorasi di bawahnya akan dieksekusi. Kode ini mengaplikasikan autentikasi JWT pada rute tersebut. Hanya pengguna yang memiliki JWT yang valid dan terkait dengan otorisasi yang sesuai yang dapat mengakses rute ini. Lokasi JWT diatur sebagai header.
```
def get_task_by_id(id):
    task = Tasks.query.filter_by(id=id).first()
```
Kode ini mendefinisikan fungsi get_task_by_id(id) sebagai penangan rute HTTP. Fungsi ini akan dieksekusi ketika rute '/int:id' dengan metode GET diakses. Kode ini menggunakan SQLAlchemy untuk mengambil tugas dari database berdasarkan ID yang diberikan. Fungsi Tasks.query digunakan untuk memulai query SQLAlchemy pada tabel 'Tasks'. Metode filter_by(id=id) digunakan untuk memfilter tugas berdasarkan kolom 'id' yang sesuai dengan nilai id yang diberikan. Metode first() digunakan untuk mengambil satu hasil pertama yang sesuai dengan query.
```
if not task:
    return jsonify({'message': 'task not found'}), 404

task = task.serialize()
return jsonify({
    'success': True,
    'data': task
}), 200
```
Kode ini memeriksa apakah tugas dengan ID yang diberikan ditemukan dalam database. Jika tidak ditemukan, maka respons JSON dengan pesan kesalahan "task not found" dikembalikan dengan kode status 404 (Not Found). Fungsi task.serialize() digunakan untuk mengonversi objek tugas menjadi format JSON. Kode ini mengembalikan respons JSON yang berisi tugas yang ditemukan dalam format JSON. Kode status 200 (OK) digunakan untuk menandakan bahwa permintaan berhasil
```
@taskBp.route('/', methods=['POST'], strict_slashes = False)
@jwt_required(locations=["headers"])
```
Kode ini mendefinisikan rute HTTP dengan metode POST dan path '/' pada Blueprint 'task'. Ketika rute ini diakses dengan metode POST, fungsi yang terdekorasi di bawahnya akan dieksekusi. Kode ini mengaplikasikan autentikasi JWT pada rute tersebut. Hanya pengguna yang memiliki JWT yang valid dan terkait dengan otorisasi yang sesuai yang dapat mengakses rute ini. Lokasi JWT diatur sebagai header.
```
def create_task():
    data = request.get_json()
    title = data['title']
    description = data['description']
    user_id = get_jwt_identity()

    if not title or not description or not user_id:
        return jsonify({"message": "Incomplete data"}), 422
    
    new_task = Tasks(title = title, description = description, user_id = user_id)

    db.session.add(new_task)
    db.session.commit()

    return jsonify({
        "success": True,
        "data": new_task.serialize()
    }), 200
```
Kode ini mendefinisikan fungsi create_task() sebagai penangan rute HTTP. Fungsi ini akan dieksekusi ketika rute '/' dengan metode POST diakses. Kode ini menggunakan request.get_json() untuk mendapatkan data dari permintaan HTTP dalam format JSON. Data ini kemudian digunakan untuk mengambil nilai 'title', 'description', dan 'user_id'. Nilai 'user_id' didapatkan dari JWT pengguna yang terautentikasi. Kode ini memeriksa apakah nilai 'title', 'description', atau 'user_id' kosong. Jika salah satu nilai tersebut kosong, maka respons JSON dengan pesan kesalahan "Incomplete data" dikembalikan dengan kode status 422 (Unprocessable Entity). Kode ini membuat objek tugas baru dengan menggunakan nilai 'title', 'description', dan 'user_id'. Objek tugas baru ini kemudian ditambahkan ke sesi database menggunakan db.session.add() dan perubahan disimpan menggunakan db.session.commit(). Kode ini mengembalikan respons JSON yang berisi tugas yang baru dibuat dalam format JSON. Kode status 200 (OK) digunakan untuk menandakan bahwa permintaan berhasil.
```
@taskBp.route('<int:id>', methods=['PUT'], strict_slashes = False)
@jwt_required(locations=["headers"])
```
Kode ini mendefinisikan rute HTTP dengan metode PUT dan path 'int:id' pada Blueprint 'task'. <int:id> menunjukkan bahwa kita mengharapkan parameter id dengan tipe data integer dalam path. Ketika rute ini diakses dengan metode PUT, fungsi yang terdekorasi di bawahnya akan dieksekusi. Kode ini mengaplikasikan autentikasi JWT pada rute tersebut. Hanya pengguna yang memiliki JWT yang valid dan terkait dengan otorisasi yang sesuai yang dapat mengakses rute ini. Lokasi JWT diatur sebagai header.
```
def update_task(id):
    current_user_id = get_jwt_identity()

    data = request.get_json()
    title = data['title']
    description = data['description']

    task = Tasks.query.filter_by(id=id).first()
```
Kode ini menggunakan get_jwt_identity() untuk mendapatkan id pengguna dari JWT yang terautentikasi. Selanjutnya, data dari permintaan HTTP dalam format JSON diambil menggunakan request.get_json(). Data ini kemudian digunakan untuk mengambil nilai 'title' dan 'description' dari permintaan. Kode ini menggunakan Tasks.query.filter_by(id=id).first() untuk mengambil tugas dari database berdasarkan id yang diberikan dalam path rute.
```
    if not task:
        return jsonify({"message": "Task Not Found!"}), 404
    
    if not title or not description:
        return jsonify({"message": "Incomplete data"}), 422
    
    if current_user_id != task.user_id:
        return jsonify({"message": "Unauthorized action"}), 422
    
    task.title = title
    task.description = description

    db.session.commit()

    return jsonify({
        "success": True,
        "message": "Task successfully updated!"
    }), 200
```
Jika tugas tidak ditemukan, maka respons JSON dengan pesan "Task Not Found!" dikembalikan dengan kode status 404 (Not Found). Kode ini memeriksa apakah nilai 'title' atau 'description' kosong. Jika salah satu nilai tersebut kosong, maka respons JSON dengan pesan kesalahan "Incomplete data" dikembalikan dengan kode status 422 (Unprocessable Entity). Selain itu, juga dilakukan pengecekan apakah pengguna yang meminta pembaruan adalah pemilik tugas. Jika bukan, maka respons JSON dengan pesan "Unauthorized action" dikembalikan dengan kode status 422 (Unprocessable Entity). Kode ini mengubah nilai 'title' dan 'description' dari objek tugas yang ditemukan sebelumnya. Perubahan ini kemudian disimpan ke dalam database menggunakan db.session.commit(). Kode ini mengembalikan respons JSON yang berisi pesan sukses "Task successfully updated!" dalam format JSON. Kode status 200 (OK) digunakan untuk menandakan bahwa pembaruan tugas berhasil.
```
@taskBp.route('/status/<int:id>', methods=['GET'], strict_slashes = False)
@jwt_required(locations=['headers'])
```
Kode ini mendefinisikan rute HTTP dengan metode GET dan path '/status/int:id' pada Blueprint 'task'. <int:id> menunjukkan bahwa kita mengharapkan parameter id dengan tipe data integer dalam path. Ketika rute ini diakses dengan metode GET, fungsi yang terdekorasi di bawahnya akan dieksekusi. Kode ini mengaplikasikan autentikasi JWT pada rute tersebut. Hanya pengguna yang memiliki JWT yang valid dan terkait dengan otorisasi yang sesuai yang dapat mengakses rute ini. Lokasi JWT diatur sebagai header.
```
def update_status(id):
    current_user_id = get_jwt_identity()
    task = Tasks.query.filter_by(id=id).first()

    if not task:
        return jsonify({'message': 'task not found'}), 422

    data = request.get_json()
    status = data.get('status')
    task.status = status
    db.session.commit()
    return jsonify({
        "success": True,
        "message": "status successfully updated"
    }), 200
```
Kode ini mendefinisikan fungsi update_status() sebagai penangan rute HTTP. Fungsi ini akan dieksekusi ketika rute '/status/int:id' dengan metode GET diakses. Kode ini menggunakan get_jwt_identity() untuk mendapatkan id pengguna dari JWT yang terautentikasi. Selanjutnya, tugas dari database diambil berdasarkan id yang diberikan dalam path rute menggunakan Tasks.query.filter_by(id=id).first(). Kode ini memeriksa apakah tugas ditemukan dalam database. Jika tugas tidak ditemukan, maka respons JSON dengan pesan "task not found" dikembalikan dengan kode status 422 (Unprocessable Entity). Kode ini menggunakan request.get_json() untuk mendapatkan data dari permintaan dalam format JSON. Selanjutnya, nilai 'status' diambil dari data JSON menggunakan data.get('status'). Status tugas kemudian diperbarui dengan nilai yang diperoleh, dan perubahan ini disimpan ke dalam database menggunakan db.session.commit(). Kode ini mengembalikan respons JSON yang berisi pesan sukses "status successfully updated" dalam format JSON. Kode status 200 (OK) digunakan untuk menandakan bahwa pembaruan status tugas berhasil.
```
@taskBp.route('<int:id>', methods=['DELETE'], strict_slashes = False)
@jwt_required(locations=["headers"])
```
Kode ini mendefinisikan rute HTTP dengan metode DELETE dan path 'int:id' pada Blueprint 'task'. <int:id> menunjukkan bahwa kita mengharapkan parameter id dengan tipe data integer dalam path. Ketika rute ini diakses dengan metode DELETE, fungsi yang terdekorasi di bawahnya akan dieksekusi. Kode ini mengaplikasikan autentikasi JWT pada rute tersebut. Hanya pengguna yang memiliki JWT yang valid dan terkait dengan otorisasi yang sesuai yang dapat mengakses rute ini. Lokasi JWT diatur sebagai header.
```
def delete_task(id):
    task = Tasks.query.filter_by(id=id).first()
    current_user_id = get_jwt_identity()
```
Kode ini mendefinisikan fungsi delete_task() sebagai penangan rute HTTP. Fungsi ini akan dieksekusi ketika rute 'int:id' dengan metode DELETE diakses. Kode ini menggunakan Tasks.query.filter_by(id=id).first() untuk mendapatkan tugas dari database berdasarkan id yang diberikan dalam path rute. Kode ini menggunakan get_jwt_identity() untuk mendapatkan id pengguna saat ini dari JWT yang terautentikasi.
```
    if not task:
        return jsonify({"message": "Task Not Found!"}), 404
    
    if current_user_id != task.user_id:
        return jsonify({"message": "Unauthorized action"}), 422
    
    db.session.delete(task)
    db.session.commit()

    return jsonify({
        "success": True,
        "message": "Task successfully deleted!"
    })
```
Kode ini memeriksa apakah tugas ditemukan dalam database. Jika tugas tidak ditemukan, maka respons JSON dengan pesan "Task Not Found!" dikembalikan dengan kode status 404 (Not Found). Kode ini memeriksa apakah id pengguna saat ini sesuai dengan id pengguna yang memiliki tugas. Jika tidak sesuai, maka respons JSON dengan pesan "Unauthorized action" dikembalikan dengan kode status 422 (Unprocessable Entity). Kode ini menggunakan db.session.delete(task) untuk menghapus tugas dari sesi database. Perubahan ini kemudian disimpan ke dalam database menggunakan db.session.commit(). Kode ini mengembalikan respons JSON yang berisi pesan sukses "Task successfully deleted!" dalam format JSON. Kode status default 200 (OK) digunakan untuk menandakan bahwa penghapusan tugas berhasil.

### Code 14
Code ini berada di app/user/__init__.py
```
from flask import Blueprint
userBp = Blueprint('user', __name__)
from app.user import routes
```
Kode ini mengimpor modul Blueprint dari pustaka Flask. Blueprint adalah objek yang digunakan untuk mengorganisir rute, fungsi, dan templat yang terkait dalam aplikasi Flask. Kode ini membuat objek Blueprint dengan nama 'user'. Parameter pertama adalah nama Blueprint, dan parameter kedua 'name' digunakan untuk menentukan nama modul Blueprint saat ini. Kode ini mengimpor modul 'routes' yang terkait dengan rute pengguna. Modul 'routes' kemungkinan besar berisi definisi rute HTTP untuk operasi yang terkait dengan pengguna, seperti registrasi, masuk, profil, dan lainnya.

### Code 15
Code ini berada di app/user/routes.py
```
@userBp.route('/', methods=['GET'], strict_slashes = False)
def get_all_users():
    limit = request.args.get('limit', 10)
    
    if type(limit) is not int:
        return jsonify({
            "message": "Invalid Parameter"
        }), 400
    
    users = db.session.execute(db.select(Users).limit(limit)).scalars()

    print(users)
```
Kode ini mendefinisikan rute '/' dengan metode HTTP GET pada Blueprint 'user'. Rute ini akan menangani permintaan untuk mendapatkan semua pengguna. Kode ini menggunakan objek request untuk mengambil nilai parameter 'limit' dari URL. Jika parameter tidak ada, maka akan digunakan nilai default 10. Kode ini memeriksa tipe data parameter 'limit'. Jika tipe datanya bukan integer, maka akan dikembalikan respons JSON dengan pesan error "Invalid Parameter" dan kode status 400. Kode ini menggunakan objek db.session untuk melakukan eksekusi query yang mengambil daftar pengguna dari tabel 'Users' dengan batasan jumlah data sesuai nilai 'limit'. Hasilnya adalah objek hasil query yang akan diperlakukan sebagai iterabel.
```
    result = []
    for user in users:
        result.append(user.serialize())

    return jsonify({
        "success": True,
        "data": result
    }), 200
```
Kode ini melakukan iterasi melalui setiap objek pengguna dalam hasil query dan mengubahnya menjadi bentuk yang dapat di-serialize. Asumsinya adalah setiap objek pengguna memiliki metode serialize() yang mengembalikan representasi serialisasi data pengguna. Kode ini mengembalikan respons JSON yang berisi data pengguna yang telah di-serialize. Respon memiliki atribut "success" yang bernilai True dan atribut "data" yang berisi daftar pengguna yang telah di-serialize. Kode status 200 menunjukkan bahwa permintaan berhasil dilakukan.
```
@userBp.route('/', methods=['POST'], strict_slashes = False)
def create_user():
    data = request.get_json()
    name = data['name']
    email = data['email']
    password = data['password']
```
Kode ini mendefinisikan rute '/' dengan metode HTTP POST pada Blueprint 'user'. Rute ini akan menangani permintaan untuk membuat pengguna baru. Kode ini menggunakan objek request untuk mendapatkan data JSON yang dikirim dengan permintaan POST. Kode ini mengambil nilai 'name', 'email', dan 'password' dari data JSON yang dikirim dengan permintaan. Asumsinya adalah data JSON memiliki kunci yang sesuai dengan nama kolom pada tabel pengguna.
```
    if not name or not email or not password:
        return jsonify({
            'message': "Incomplete data"
        }), 422
    
    new_user = Users(name = name, email = email, password = password)

    db.session.add(new_user)
    db.session.commit()

    return jsonify({
        "success": True,
        "data": new_user.serialize()
    }), 200
```
Kode ini memeriksa apakah semua nilai yang diperlukan ('name', 'email', dan 'password') telah disediakan. Jika ada nilai yang kosong, maka akan dikembalikan respons JSON dengan pesan error "Incomplete data" dan kode status 422. Kode ini membuat objek pengguna baru dengan menggunakan nilai-nilai yang diterima dari permintaan. Asumsinya adalah terdapat kelas Users yang menggambarkan model pengguna dan memiliki konstruktor yang menerima nilai-nilai tersebut. Kode ini menggunakan objek db.session untuk menambahkan objek pengguna baru ke dalam sesi database dan melakukan commit untuk menyimpan perubahan ke dalam database. Kode ini mengembalikan respons JSON yang berisi data pengguna baru yang telah di-serialize. Respon memiliki atribut "success" yang bernilai True dan atribut "data" yang berisi data pengguna baru yang telah di-serialize. Kode status 200 menunjukkan bahwa permintaan berhasil dilakukan.
```
@userBp.route('<int:id>', methods=['PUT'], strict_slashes = False)
def update_user(id):
    data = request.get_json()
    name = data['name']
    email = data['email']
    password = data['password']

    user = Users.query.filter_by(id=id).first()
```
Kode ini mendefinisikan rute dengan pola URL <int:id>, yang berarti akan menangani permintaan PUT untuk memperbarui pengguna dengan ID tertentu. Blueprint yang digunakan adalah userBp. Kode ini menggunakan objek request untuk mendapatkan data JSON yang dikirim dengan permintaan PUT. Kode ini mengambil nilai 'name', 'email', dan 'password' dari data JSON yang dikirim dengan permintaan. Asumsinya adalah data JSON memiliki kunci yang sesuai dengan nama kolom pada tabel pengguna. Kode ini menggunakan metode filter_by pada kelas Users (asumsi kelas ini merupakan model pengguna) untuk mencari pengguna dengan ID yang sesuai dengan nilai id yang diberikan. Jika pengguna tidak ditemukan, maka akan dikembalikan respons JSON dengan pesan error "User Not Found!" dan kode status 404.
```
    if not user:
        return jsonify({
            "message": 'User Not Found!'
        }), 404
    
    if not name or not email or not password:
        return jsonify({
            'message': "Incomplete data"
        }), 422
    
    user.name = name
    user.email = email
    user.password = password
    db.session.commit()

    return jsonify({
        "success": True,
        "message": "User successfully updated"
    }), 200
```
Kode ini memeriksa apakah semua nilai yang diperlukan ('name', 'email', dan 'password') telah disediakan. Jika ada nilai yang kosong, maka akan dikembalikan respons JSON dengan pesan error "Incomplete data" dan kode status 422. Kode ini memperbarui nilai-nilai pengguna yang ditemukan berdasarkan ID. Nilai-nilai tersebut diubah sesuai dengan nilai yang diterima dari permintaan. Setelah itu, perubahan tersebut dicommit ke dalam database menggunakan db.session.commit(). Kode ini mengembalikan respons JSON dengan atribut "success" yang bernilai True dan atribut "message" yang menyatakan bahwa pengguna telah berhasil diperbarui. Kode status 200 menunjukkan bahwa permintaan berhasil dilakukan.
```
@userBp.route('<int:id>', methods=['DELETE'], strict_slashes = False)
def delete_user(id):
    user = Users.query.filter_by(id=id).first()
```
Kode ini mendefinisikan rute dengan pola URL <int:id>, yang berarti akan menangani permintaan DELETE untuk menghapus pengguna dengan ID tertentu. Blueprint yang digunakan adalah userBp. Kode ini menggunakan metode filter_by pada kelas Users (asumsi kelas ini merupakan model pengguna) untuk mencari pengguna dengan ID yang sesuai dengan nilai id yang diberikan. Jika pengguna tidak ditemukan, maka akan dikembalikan respons JSON dengan pesan error "User Not Found!" dan kode status 404.
    if not user:
        return jsonify({
            "message": 'User Not Found!'
        }), 404
    
    db.session.delete(user)
    db.session.commit()

    return jsonify({
        "success": True,
        "message": "User successfully deleted!"
    }), 200
```
Kode ini menghapus pengguna yang ditemukan dari database menggunakan db.session.delete(user). Setelah itu, perubahan tersebut dicommit ke dalam database menggunakan db.session.commit(). Kode ini mengembalikan respons JSON dengan atribut "success" yang bernilai True dan atribut "message" yang menyatakan bahwa pengguna telah berhasil dihapus. Kode status 200 menunjukkan bahwa permintaan berhasil dilakukan.
```

@userBp.route('<int:id>/tasks', methods=['GET'], strict_slashes = False)
def get_user_task(id):
    limit = request.args.get('limit', 10)
    
    if type(limit) is not int:
        return jsonify({
            "message": "Invalid Parameter"
        }), 400
```
Kode ini mendefinisikan rute dengan pola URL <int:id>/tasks, yang berarti akan menangani permintaan GET untuk mendapatkan tugas-tugas yang terkait dengan pengguna berdasarkan ID pengguna. Blueprint yang digunakan adalah userBp. Kode ini menggunakan objek request dari Flask untuk mengambil nilai parameter limit dari argumen URL. Jika parameter tidak ditemukan, maka nilai default yang digunakan adalah 10. Kode ini memeriksa apakah tipe data limit adalah integer. Jika bukan, maka akan dikembalikan respons JSON dengan pesan error "Invalid Parameter" dan kode status 400.
```
    tasks = Tasks.query.filter_by(user_id = id).all()

    if not tasks:
        return jsonify({"message": "Tasks Not Found!"}), 404
    
    result = []
    for task in tasks:
        result.append(task.serialize())

    return jsonify({
        "success": True,
        "data": result
    }), 200
```
Kode ini menggunakan metode filter_by pada kelas Tasks (asumsi kelas ini merupakan model tugas) untuk mencari tugas-tugas yang memiliki user_id sesuai dengan nilai id yang diberikan. Hasilnya disimpan dalam variabel tasks. Kode ini memeriksa apakah tasks kosong. Jika ya, maka akan dikembalikan respons JSON dengan pesan error "Tasks Not Found!" dan kode status 404. Kode ini mengiterasi setiap tugas dalam tasks dan mengonversi setiap tugas menjadi format serialisasi menggunakan metode serialize(). Hasilnya disimpan dalam variabel result. Selanjutnya, dilakukan pengembalian respons JSON yang berisi atribut "success" yang bernilai True dan atribut "data" yang berisi tugas-tugas yang telah di-serialize. Kode status 200 menunjukkan bahwa permintaan berhasil dilakukan.

### Code 16
Code ini berada di app/__init__.py
```
def create_app(config_class = Config):
    app = Flask(__name__)
    app.config.from_object(config_class)
    db.init_app(app)
    migrate.init_app(app, db)

    jwt.init_app(app)
```
Kode ini membuat objek Flask dengan nama aplikasi yang sesuai dengan modul saat ini. Kode ini mengambil konfigurasi aplikasi dari kelas config_class. Kelas konfigurasi ini biasanya digunakan untuk mengatur variabel-variabel konfigurasi, seperti pengaturan database, kunci rahasia, atau pengaturan lain yang diperlukan oleh aplikasi. Kode ini menginisialisasi ekstensi Flask-SQLAlchemy dengan objek aplikasi app. Ekstensi ini digunakan untuk berinteraksi dengan database menggunakan ORM SQLAlchemy. Kode ini menginisialisasi ekstensi Flask-Migrate dengan objek aplikasi app dan objek database db. Ekstensi ini digunakan untuk melakukan migrasi skema database. 
```
    jwt.init_app(app)
    
    app.register_blueprint(taskBp, url_prefix='/api/tasks')
    app.register_blueprint(userBp, url_prefix='/api/users')
    app.register_blueprint(authBp, url_prefix='/api/auth')
    app.register_blueprint(frontendBp, url_prefix = "/")

    return app
```
Kode ini menginisialisasi ekstensi Flask-JWT-Extended dengan objek aplikasi app. Ekstensi ini digunakan untuk mengimplementasikan otentikasi dan otorisasi berbasis JWT (JSON Web Tokens). Kode ini mendaftarkan blueprint untuk setiap bagian aplikasi, seperti tugas (tasks), pengguna (users), otentikasi (auth), dan tampilan frontend. Blueprint digunakan untuk mengatur rute dan logika terkait untuk setiap bagian aplikasi secara terpisah. Kode ini mengembalikan objek aplikasi Flask yang telah dikonfigurasi dan siap digunakan sebagai aplikasi web yang berjalan.

### Code 17
Code ini berada di app/extensions.py
```
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_jwt_extended import JWTManager

db = SQLAlchemy()
migrate = Migrate()
jwt = JWTManager()
```
db = SQLAlchemy(): Menginisialisasi objek SQLAlchemy yang digunakan untuk berinteraksi dengan database menggunakan ORM (Object-Relational Mapping). SQLAlchemy mempermudah pemodelan data dan kueri database dalam aplikasi Flask.

migrate = Migrate(): Menginisialisasi objek Migrate yang digunakan untuk melakukan migrasi skema database. Flask-Migrate bekerja dengan SQLAlchemy untuk mengelola versi skema dan menerapkan perubahan skema yang terkait dengan model aplikasi.

jwt = JWTManager(): Menginisialisasi objek JWTManager yang digunakan untuk mengelola otentikasi dan otorisasi berbasis JWT (JSON Web Tokens). Flask-JWT-Extended menyediakan fitur-fitur yang kuat untuk mengamankan rute-rute aplikasi dengan token dan melaksanakan logika otorisasi yang fleksibel.
