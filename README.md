# Lab11Web - Pratikum PHP OOP Lanjutan

Nama: Marsya Nabila Putri

NIM: 312410338

Kelas: TI.24.A4

Matakuliah: Pemograman Web 1



## Tujuan Pratikum
1. Mahasiswa memahami konsep dasar Framework Modular.
   
2. Mahasiswa memahami konsep dasar Routing pada PHP.
   
3. Mahasiswa mampu membuat mini-framework sederhana menggunakan PHP OOP.

## .htacces

```
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /lab11_php_oop/

    # Jangan proses jika file/folder aslinya memang ada
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f

    # Arahkan semua request lain ke index.php
    RewriteRule ^(.*)$ index.php/$1 [L]
</IfModule>
```

`.htaccess` berfungsi untuk mengaktifkan modul `mod_rewrite` agar URL bisa diubah. Bagian `RewriteEngine On` menyalakan fitur pengaturan URL. `RewriteBase /lab11_php_oop/` memberi tahu server bahwa aturan ini berlaku di folder project. Dua baris kondisi setelahnya memastikan bahwa jika yang diminta adalah file atau folder asli, server tidak mengubahnya. Aturan terakhir mengarahkan semua URL yang bukan file asli menuju `index.php`, sehingga sistem routing bisa membaca bagian URL seperti `artikel/tambah` tanpa memakai `.php`.


## config.php

```php
<?php
$config = [
    'host' => 'localhost',
    'username' => 'Marsya Nabila',
    'password' => '290306',
    'db_name' => 'latihan_oop'
];
```


Kode tersebut berisi pengaturan agar aplikasi bisa terhubung ke database MySQL. Bagian `'host' => 'localhost'` menunjukkan bahwa database berada di komputer yang sama. `'username' => 'Marsya Nabila'` adalah nama pengguna MySQL yang kamu pakai untuk login. `'password' => '290306'` adalah kata sandi MySQL milik akun tersebut. `'db_name' => 'latihan_oop'` adalah nama database yang akan dipakai aplikasi. File ini nanti dibaca oleh class Database untuk membuat koneksi ke MySQL.


## index.php

```php
<?php
include "config.php";
include "class/Database.php";
include "class/Form.php";

session_start();
$path = isset($_SERVER['PATH_INFO']) ? $_SERVER['PATH_INFO'] : '/home/index';
$segments = explode('/', trim($path, '/'));

$mod = isset($segments[0]) && $segments[0] !== '' ? $segments[0] : 'home';
$page = isset($segments[1]) && $segments[1] !== '' ? $segments[1] : 'index';

$file = "module/{$mod}/{$page}.php";

include "template/header.php";
include "template/sidebar.php";

if (file_exists($file)) {
    include $file;
} else {
    echo "<div style='padding:20px;'><h3>Halaman tidak ditemukan: {$mod}/{$page}</h3></div>";
}

include "template/footer.php";
```

File `index.php` berfungsi sebagai router utama dalam project. File ini mengambil URL yang diakses pengguna melalui PATH_INFO, lalu memecahnya menjadi dua bagian: nama modul dan nama halaman. Dari dua nilai tersebut, sistem menentukan file mana yang harus dimuat dari folder module/. Jika file sesuai rute ditemukan, maka kontennya ditampilkan. Jika tidak, sistem akan menampilkan pesan bahwa halaman tidak tersedia. File ini juga memuat template header, sidebar, dan footer sehingga setiap halaman memiliki tampilan yang konsisten di seluruh aplikasi.

## Class/ (database.php)

```php
<?php
class Database {

    protected $host;
    protected $user;
    protected $password;
    protected $db_name;
    protected $conn;

    public function __construct() {
        $this->getConfig();

        $this->conn = new mysqli($this->host, $this->user, $this->password, $this->db_name);

        if ($this->conn->connect_error) {
            die("DB Error: " . $this->conn->connect_error);
        }

        $this->conn->set_charset("utf8");
    }

    private function getConfig() {
        include __DIR__ . '/../config.php';

        $this->host = $config['host'];
        $this->user = $config['username'];
        $this->password = $config['password'];
        $this->db_name = $config['db_name'];
    }

    public function fetchAll($sql) {
        $res = $this->conn->query($sql);

        $data = [];
        while ($row = $res->fetch_assoc()) {
            $data[] = $row;
        }

        return $data;
    }

    public function insert($table, $data) {
        $cols = implode("`, `", array_keys($data));
        $vals = array_map([$this->conn, 'real_escape_string'], array_values($data));
        $vals = "'" . implode("','", $vals) . "'";

        $sql = "INSERT INTO `$table` (`$cols`) VALUES ($vals)";
        return $this->conn->query($sql);
    }
}
?>
```

File Database.php berfungsi sebagai class untuk menangani koneksi dan operasi dasar ke database MySQL. Pada saat class di-inisialisasi, file ini mengambil konfigurasi database dari config.php, kemudian membuat koneksi menggunakan mysqli. Jika koneksi gagal, program akan berhenti dan menampilkan pesan error. Class ini juga mengatur charset menjadi UTF-8 agar data teks tersimpan dengan benar.

Di dalamnya terdapat fungsi fetchAll() yang digunakan untuk menjalankan sebuah query dan mengambil semua data hasilnya dalam bentuk array. Selain itu, terdapat fungsi insert() yang mempermudah proses penyimpanan data ke dalam tabel. Fungsi ini secara otomatis membuat query INSERT berdasarkan nama kolom dan nilai yang diberikan. Dengan class ini, program dapat melakukan operasi database lebih sederhana dan terstruktur dalam konsep OOP.

## Class/ (form.php)

```php
<?php
class Form {
    private $fields = [];
    private $action;
    private $submit = "Submit Form";
    private $jumField = 0;

    public function __construct($action = "", $submit = "Submit Form") {
        $this->action = $action;
        $this->submit = $submit;
    }

    public function displayForm() {
        echo "<form action='" . $this->action . "' method='POST' enctype='multipart/form-data'>";
        echo '<table width="100%" border="0">';
        foreach ($this->fields as $field) {
            echo "<tr><td align='right' valign='top' style='width:180px;'>" . $field['label'] . "</td>";
            echo "<td>";
            switch ($field['type']) {
                case 'textarea':
                    $val = isset($field['value']) ? htmlspecialchars($field['value']) : '';
                    echo "<textarea name='" . $field['name'] . "' cols='30' rows='4'>{$val}</textarea>";
                    break;
                case 'select':
                    echo "<select name='" . $field['name'] . "'>";
                    foreach ($field['options'] as $value => $label) {
                        $selected = (isset($field['value']) && $field['value'] == $value) ? "selected" : "";
                        echo "<option value='" . htmlspecialchars($value) . "' {$selected}>" . $label . "</option>";
                    }
                    echo "</select>";
                    break;
                case 'radio':
                    foreach ($field['options'] as $value => $label) {
                        $checked = (isset($field['value']) && $field['value'] == $value) ? "checked" : "";
                        echo "<label><input type='radio' name='" . $field['name'] . "' value='" . htmlspecialchars($value) . "' {$checked}> " . $label . "</label> ";
                    }
                    break;
                case 'checkbox':
                    foreach ($field['options'] as $value => $label) {
                        $checked = "";
                        if (isset($field['value']) && is_array($field['value']) && in_array($value, $field['value'])) $checked = "checked";
                        echo "<label><input type='checkbox' name='" . $field['name'] . "[]' value='" . htmlspecialchars($value) . "' {$checked}> " . $label . "</label> ";
                    }
                    break;
                case 'password':
                    echo "<input type='password' name='" . $field['name'] . "'>";
                    break;
                case 'file':
                    echo "<input type='file' name='" . $field['name'] . "'>";
                    break;
                default:
                    $val = isset($field['value']) ? htmlspecialchars($field['value']) : '';
                    echo "<input type='text' name='" . $field['name'] . "' value='{$val}'>";
                    break;
            }
            echo "</td></tr>";
        }
        echo "<tr><td colspan='2' style='text-align:right; padding-top:10px;'><input type='submit' value='" . $this->submit . "'></td></tr>";
        echo "</table>";
        echo "</form>";
    }

    public function addField($name, $label, $type = "text", $options = [], $value = null) {
        $this->fields[$this->jumField]['name'] = $name;
        $this->fields[$this->jumField]['label'] = $label;
        $this->fields[$this->jumField]['type'] = $type;
        $this->fields[$this->jumField]['options'] = $options;
        $this->fields[$this->jumField]['value'] = $value;
        $this->jumField++;
    }
}
```

File `Form.php` ini berfungsi sebagai class pembuat form dinamis. Di dalam class terdapat mekanisme untuk menambahkan field input secara fleksibel seperti text, textarea, select, radio, checkbox, password, dan file upload. Setiap field yang ditambahkan melalui fungsi addField() disimpan ke dalam array, kemudian seluruh field tersebut ditampilkan melalui fungsi displayForm().

Class ini menangani berbagai jenis input dengan pengecekan tipe pada saat form dirender. Jika field memiliki nilai sebelumnya (misalnya untuk form edit), nilainya otomatis diisi kembali. Dengan cara ini, program tidak perlu membuat form satu per satu secara manual, karena seluruh struktur formulir dibuat secara otomatis berdasarkan konfigurasi yang diberikan melalui class ini. Class ini menjadikan pembuatan form lebih rapi, reusable, dan sesuai konsep OOP.
