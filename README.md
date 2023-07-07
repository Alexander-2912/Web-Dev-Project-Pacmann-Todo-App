# Web-Development

## Project Objective
Objektif dari projek ini adalah membuat sebuah website yang dapat digunakan untuk menyimpan list-list suatu kegiatan, barang, maupun yang lainnya. Dalam pembuatannya, terdapat fitur-fitur dan batasan yang ada untuk diikuti. Hasil akhir dari project ini adalah terciptanya sebuah website yang responsif, dinamis, dan dapat digunakan dengan baik.

## Functions
Dalam projek ini terdapat banyak folder, file, dan code digunakan. Dokumentasi ini akan menjelaskan seluruh code yang digunakan secara singkat.

### Code 1
Code terdapat pada app/auth/__init__.py
'''
from flask import Blueprint
from flask_cors import CORS

authBp = Blueprint('auth', __name__)
CORS(authBp)

from app.auth import routes
'''
Code ini terdari import Blueprint dan CORS agar dapat menggunakan fungsi yang disediakan dari Flask dan Flask-CORS. Kemudian 'authBp' digunakan sebagai objek Blueprint agar bisa menggunakan fungsi Blueprint yang disediakan Flask, Blueprint diberi nama 'auth'. Setelah itu CORS diterapkan menggunakan objek 'authBp' yang telah dibuat sebelumnya. 

### Code 2
Code tedapat pada app/auth/routes.py
'''
from flask import request, jsonify
from werkzeug.security import generate_password_hash, check_password_hash
from flask_jwt_extended import create_access_token, create_refresh_token, jwt_required, get_jwt_identity, get_jwt
from sqlalchemy.exc import IntegrityError

from app.models.blacklist_token import BlacklistToken
from app.models.user import Users
from app.extension import db, jwt
from app.auth import authBp
'''
Code ini digunakan untuk melakukan import dari beberapa module agar dapat menggunakan fungsi yang ada pada module tersebut.

'''
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

'''
Kode yang diberikan adalah penggunaan Blueprint dalam Flask untuk menangani rute '/register/' dan method 'POST'. Fungsi register() dihubungkan ke rute '/register' pada blueprint 'authBp'. Dalam fungsi register(), pertama-tama akan mengambil data JSON menggunakan 'register.get_json()', data yang diambil data 'name', 'email', dan 'password'.
Setelah mendapatkan data, akan dilakukan pemeriksaan terhadap ketersediaan data yang ada. Jika ada data yang kosong, makan akan dikembalikan pesan error dan status kode 400.
Jika data sudah terisi semua, makan data pengguna baru akan dimasukkan kedalam database. Jika terdapat kesalahan 'IntegrityError', itu menyatakan bahwa nama user sudah tersedia, dan akan dikembalikan pesan bahwa nama sudah terdaftar dan status kode 422
Jika tidak ada kesalahan, pengguna baru berhasil ditambahkan kedalam database. Sebuah respon akan dikembalikan dan status kode 200

'''
@authBp.route('/login', methods=['POST'], strict_slashes=False)
def login():
    data = request.get_json()

    email = data.get('email', None)
    password = data.get('password', None)

    # Fields are required
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
'''
Kode yang diberikan adalah penggunaan Blueprint dalam Flask untuk menangani rute '/login/' dan method 'POST'. Fungsi login() dihubungkan ke rute '/login' pada blueprint 'authBp'. Dalam fungsi login(), pertama-tama akan mengambil data JSON menggunakan 'register.get_json()', data yang diambil data 'email', dan 'password'.
Setelah mendapatkan data, akan dilakukan pemeriksaan terhadap ketersediaan data yang ada. Jika ada data yang kosong, makan akan dikembalikan pesan error dan status kode 400.
Setelah memastikan data sudah diisi, maka selanjutnya akan dilakukan pemeriksaan di database apakah email dan password sudah sesuai. Jika pengguna tidak ditemukan, maka akan dikembalikan respon berupa pesan dan status kode 422
Jika tidak ada kesalahan, maka akses token akan dibuat dengan refresh token menggunakan fungsi 'create_access_token()' dan 'create_refresh_token()'

'''
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
'''
Fungsi 'refresh_token()' dihubungkan ke route /refresh pada blueprint authBp dengan method 'POST'. Didalam fungsi terdapat @jwt_required(refresh=True) untuk memastikan hanya pengguna yang memiliki refresh token yang valid dapat mengaksesnya. Di dalam fungsi, kita mendapatkan identitas pengguna dari refresh token yang valid menggunakan 'get_jwt_identity()'. Kemudian, kita membuat akses token baru menggunakan 'create_access_token()' dengan identitas pengguna yang diperoleh. Akses token baru tersebut dikembalikan dalam respons JSON dengan status kode 200. Tujuan dari fungsi ini adalah untuk memberikan akses token baru kepada pengguna setelah refresh token mereka divalidasi.
Fungsi 'logout()' dihubungkan ke route '/logout' pada blueprint authBp dengan method 'POST'. Didalam fungsi terdapat decorator @jwt_required(locations=['headers']) untuk memastikan hanya pengguna yang mengirimkan token akses dalam header permintaan yang dapat melakukan logout. Di dalam fungsi, kita mendapatkan informasi token yang digunakan dalam permintaan menggunakan get_jwt(). Selanjutnya, kita mengambil "jti" (JWT ID) dari token tersebut dan membuat objek BlacklistToken dengan menggunakan nilai "jti". Objek BlacklistToken kemudian ditambahkan ke database untuk memasukkan token ke dalam daftar token yang diambil. Setelah itu, perubahan pada basis data dikonfirmasi dengan melakukan db.session.commit(). Terakhir, sebuah respons JSON dikembalikan dengan pesan bahwa logout berhasil dan status kode 200. Tujuan dari fungsi ini adalah untuk mencabut token akses yang digunakan sehingga token tersebut tidak lagi valid dan tidak dapat digunakan untuk mengakses sumber daya terproteksi.
Fungsi 'check_if_token_is_revoked()' adalah fungsi yang digunakan untuk menentukan apakah suatu token akses ditarik atau dicabut. Fungsi ini ditentukan dengan '@jwt.token_in_blocklist_loader' untuk JWT manager yang digunakan dalam aplikasi. Di dalam fungsi, kita mengambil "jti" (JWT ID) dari JWT payload dan mencari token dengan "jti" tersebut dalam database menggunakan BlacklistToken. Jika token ditemukan, berarti token tersebut telah ditarik dan fungsi ini mengembalikan nilai True. Jika token tidak ditemukan, berarti token masih valid dan fungsi ini mengembalikan nilai False. Dengan menggunakan fungsi ini, kita dapat memastikan bahwa token akses yang ditarik atau dicabut akan dianggap tidak valid saat melakukan verifikasi otentikasi.

