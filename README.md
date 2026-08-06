belajar_html
CREATE DATABASE IF NOT EXISTS sistem_gudang;
USE sistem_gudang;

-- Tabel akun admin
CREATE TABLE IF NOT EXISTS admin (
  id INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(50) NOT NULL UNIQUE,
  password VARCHAR(255) NOT NULL
);

-- Tabel data barang
CREATE TABLE IF NOT EXISTS barang (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nama_barang VARCHAR(100) NOT NULL,
  kategori VARCHAR(50) NOT NULL,
  stok INT NOT NULL,
  harga_beli DECIMAL(12,2) NOT NULL,
  harga_jual DECIMAL(12,2) NOT NULL,
  tanggal_masuk DATE NOT NULL
);

-- Masukkan akun admin default
INSERT INTO admin (username, password) VALUES 
('admin', MD5('admin123'));

<?php
session_start();
$host = "localhost";
$user = "root";
$pass = "";
$db   = "sistem_gudang";

$koneksi = mysqli_connect($host, $user, $pass, $db);

if (!$koneksi) {
  die("Koneksi gagal: " . mysqli_connect_error());
}

// Cek apakah sudah login
function cek_login() {
  if (!isset($_SESSION['login'])) {
    header("Location: login.php");
    exit;
  }
}
?>
<?php
include 'config.php';
$pesan = "";

if (isset($_POST['login'])) {
  $user = $_POST['username'];
  $pass = MD5($_POST['password']);

  $cek = mysqli_query($koneksi, "SELECT * FROM admin WHERE username='$user' AND password='$pass'");
  if (mysqli_num_rows($cek) == 1) {
    $_SESSION['login'] = true;
    $_SESSION['username'] = $user;
    header("Location: index.php");
  } else {
    $pesan = "Username atau Password salah!";
  }
}
?>
<!DOCTYPE html>
<html>
<head>
  <title>Login Admin</title>
  <style>
    * { font-family: Arial; box-sizing: border-box; }
    .login-box { width: 350px; margin: 120px auto; border: 1px solid #ccc; padding: 25px; border-radius: 8px; }
    h2 { text-align: center; color: #2c3e50; }
    input { width: 100%; padding: 10px; margin: 8px 0; border: 1px solid #ddd; border-radius: 4px; }
    button { width: 100%; padding: 10px; background: #3498db; color: white; border: none; border-radius: 4px; cursor: pointer; }
    .pesan { color: red; text-align: center; margin-bottom: 10px; }
  </style>
</head>
<body>
  <div class="login-box">
    <h2>Login Admin</h2>
    <?php if ($pesan) echo "<div class='pesan'>$pesan</div>"; ?>
    <form method="POST">
      <input type="text" name="username" placeholder="Username" required>
      <input type="password" name="password" placeholder="Password" required>
      <button type="submit" name="login">Masuk</button>
    </form>
    <p style="text-align:center; font-size:13px; margin-top:15px;">Akun default: admin / admin123</p>
  </div>
</body>
</html>
<?php include 'config.php'; cek_login(); ?>
<!DOCTYPE html>
<html>
<head>
  <title>Dashboard Admin</title>
  <style>
    * { margin: 0; padding: 0; font-family: Arial; }
    .sidebar { width: 200px; background: #2c3e50; height: 100vh; position: fixed; padding-top: 20px; }
    .sidebar a { display: block; color: white; padding: 12px; text-decoration: none; padding-left: 20px; }
    .sidebar a:hover { background: #34495e; }
    .main { margin-left: 200px; padding: 20px; }
    .header { background: #3498db; color: white; padding: 15px; text-align: right; }
    .box { display: inline-block; width: 28%; padding: 20px; margin: 10px; color: white; border-radius: 5px; }
    .box1 { background: #27ae60; } .box2 { background: #f39c12; } .box3 { background: #e74c3c; }
  </style>
</head>
<body>

<div class="sidebar">
  <h3 style="color:white; text-align:center; margin-bottom:20px;">ADMIN</h3>
  <a href="index.php">Dashboard</a>
  <a href="tambah_barang.php">Input Barang</a>
  <a href="data_barang.php">Data Barang</a>
  <a href="laporan.php">Laporan</a>
  <a href="logout.php" style="background:#c0392b;">Keluar</a>
</div>

<div class="header">
  Selamat datang, <?= $_SESSION['username'] ?>
</div>

<div class="main">
  <h2>Dashboard Utama</h2>
  <hr style="margin:15px 0;">

  <?php
  $total = mysqli_fetch_assoc(mysqli_query($koneksi, "SELECT COUNT(*) as total FROM barang"))['total'];
  $hari_ini = mysqli_fetch_assoc(mysqli_query($koneksi, "SELECT COUNT(*) as total FROM barang WHERE tanggal_masuk=CURDATE()"))['total'];
  $bulan_ini = mysqli_fetch_assoc(mysqli_query($koneksi, "SELECT COUNT(*) as total FROM barang WHERE MONTH(tanggal_masuk)=MONTH(CURDATE())"))['total'];
  ?>

  <div class="box box1">
    <h3>Total Barang</h3>
    <h1><?= $total ?></h1>
  </div>
  <div class="box box2">
    <h3>Barang Masuk Hari Ini</h3>
    <h1><?= $hari_ini ?></h1>
  </div>
  <div class="box box3">
    <h3>Barang Masuk Bulan Ini</h3>
    <h1><?= $bulan_ini ?></h1>
  </div>
</div>

</body>
</html>
<?php include 'config.php'; cek_login(); 

if (isset($_POST['simpan'])) {
  $nama = $_POST['nama_barang'];
  $kat  = $_POST['kategori'];
  $stok = $_POST['stok'];
  $beli = $_POST['harga_beli'];
  $jual = $_POST['harga_jual'];
  $tgl  = $_POST['tanggal_masuk'];

  mysqli_query($koneksi, "INSERT INTO barang VALUES('', '$nama', '$kat', '$stok', '$beli', '$jual', '$tgl')");
  echo "<script>alert('Barang berhasil ditambahkan!'); location.href='data_barang.php'</script>";
}
?>
<!DOCTYPE html>
<html>
<head>
  <title>Tambah Barang</title>
  <style>
    <?php include 'index.php'; ?>
    form { width: 500px; margin-top:20px; }
    input, select { width: 100%; padding:10px; margin:8px 0; border:1px solid #ddd; border-radius:4px; }
    button { padding:10px 20px; background:#27ae60; color:white; border:none; border-radius:4px; cursor:pointer; }
  </style>
</head>
<body>
<?php include 'index.php'; ?>
<div class="main">
  <h2>Tambah Data Barang</h2><hr>
  <form method="POST">
    <label>Nama Barang</label>
    <input type="text" name="nama_barang" required>
    <label>Kategori</label>
    <input type="text" name="kategori" required>
    <label>Stok</label>
    <input type="number" name="stok" required>
    <label>Harga Beli</label>
    <input type="number" name="harga_beli" required>
    <label>Harga Jual</label>
    <input type="number" name="harga_jual" required>
    <label>Tanggal Masuk</label>
    <input type="date" name="tanggal_masuk" required>
    <button type="submit" name="simpan">Simpan Barang</button>
  </form>
</div>
</body>
</html>
<?php include 'config.php'; cek_login(); ?>
<!DOCTYPE html>
<html>
<head>
  <title>Data Barang</title>
  <style>
    <?php include 'index.php'; ?>
    table { width: 100%; border-collapse: collapse; margin-top:20px; }
    th, td { border:1px solid #ddd; padding:10px; text-align:left; }
    th { background:#f1f1f1; }
  </style>
</head>
<body>
<?php include 'index.php'; ?>
<div class="main">
  <h2>Data Semua Barang</h2><hr>
  <table>
    <tr>
      <th>No</th>
      <th>Nama Barang</th>
      <th>Kategori</th>
      <th>Stok</th>
      <th>Harga Beli</th>
      <th>Harga Jual</th>
      <th>Tanggal Masuk</th>
    </tr>
    <?php
    $data = mysqli_query($koneksi, "SELECT * FROM barang ORDER BY tanggal_masuk DESC");
    $no = 1;
    while ($row = mysqli_fetch_assoc($data)) {
    ?>
    <tr>
      <td><?= $no++ ?></td>
      <td><?= $row['nama_barang'] ?></td>
      <td><?= $row['kategori'] ?></td>
      <td><?= $row['stok'] ?></td>
      <td>Rp <?= number_format($row['harga_beli'],0,',','.') ?></td>
      <td>Rp <?= number_format($row['harga_jual'],0,',','.') ?></td>
      <td><?= $row['tanggal_masuk'] ?></td>
    </tr>
    <?php } ?>
  </table>
</div>
</body>
</html>
<?php include 'config.php'; cek_login(); 
$filter = isset($_GET['filter']) ? $_GET['filter'] : 'harian';
$judul = "Laporan Harian";
$where = "WHERE tanggal_masuk = CURDATE()";

if ($filter == 'bulanan') {
  $judul = "Laporan Bulanan";
  $where = "WHERE MONTH(tanggal_masuk) = MONTH(CURDATE()) AND YEAR(tanggal_masuk) = YEAR(CURDATE())";
} elseif ($filter == 'tahunan') {
  $judul = "Laporan Tahunan";
  $where = "WHERE YEAR(tanggal_masuk) = YEAR(CURDATE())";
}
?>
<!DOCTYPE html>
<html>
<head>
  <title>Laporan Barang</title>
  <style>
    <?php include 'index.php'; ?>
    .tombol { margin:10px 5px; padding:8px 15px; text-decoration:none; color:white; border-radius:4px; }
    .h { background:#3498db; } .b { background:#f39c12; } .t { background:#27ae60; }
    table { width: 100%; border-collapse: collapse; margin-top:20px; }
    th, td { border:1px solid #ddd; padding:10px; text-align:left; }
  </style>
</head>
<body>
<?php include 'index.php'; ?>
<div class="main">
  <h2><?= $judul ?></h2><hr>
  <a href="?filter=harian" class="tombol h">Harian</a>
  <a href="?filter=bulanan" class="tombol b">Bulanan</a>
  <a href="?filter=tahunan" class="tombol t">Tahunan</a>

  <table>
    <tr>
      <th>No</th>
      <th>Nama Barang</th>
      <th>Kategori</th>
      <th>Stok</th>
      <th>Tanggal Masuk</th>
    </tr>
    <?php
    $data = mysqli_query($koneksi, "SELECT * FROM barang $where ORDER BY tanggal_masuk DESC");
    $no = 1;
    while ($row = mysqli_fetch_assoc($data)) {
    ?>
    <tr>
      <td><?= $no++ ?></td>
      <td><?= $row['nama_barang'] ?></td>
      <td><?= $row['kategori'] ?></td>
      <td><?= $row['stok'] ?></td>
      <td><?= $row['tanggal_masuk'] ?></td>
    </tr>
    <?php } ?>
  </table>
</div>
</body>
</html>
<?php
include 'config.php';
session_destroy();
header("Location: login.php");
?>