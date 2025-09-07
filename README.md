<?php

session_start();

// Recuperar datos previos y errores si existen
$old = isset($_SESSION['old']) ? $_SESSION['old'] : [];
$errors = isset($_SESSION['errors']) ? $_SESSION['errors'] : [];

// Limpiar datos de sesión para el próximo intento
unset($_SESSION['old'], $_SESSION['errors']);

// Mostrar mensajes flash
function flash_get($type) {
    if (isset($_SESSION[$type])) {
        $msg = $_SESSION[$type];
        unset($_SESSION[$type]);
        return $msg;
    }
    return '';
}
function flash_set($type, $msg) {
    $_SESSION[$type] = $msg;
}

// Verifica si ya existe un estudiante con el documento dado
function estudiantes_existe_documento($documento) {
    // Aquí deberías consultar la base de datos o el almacenamiento de estudiantes
    // Por ejemplo, usando PDO:
    // $pdo = new PDO(...);
    // $stmt = $pdo->prepare("SELECT COUNT(*) FROM estudiantes WHERE documento = ?");
    // $stmt->execute([$documento]);
    // return $stmt->fetchColumn() > 0;

    // Temporal: siempre retorna false (no existe)
    return false;
}

// Función para crear un estudiante (placeholder)
function estudiantes_create($in) {
    // Aquí deberías guardar el estudiante en la base de datos
    // Por ejemplo, usando PDO:
    // $pdo = new PDO(...);
    // $stmt = $pdo->prepare("INSERT INTO estudiantes (...) VALUES (...)");
    // $stmt->execute([...]);
    // Temporal: no hace nada
    return true;
}

// Definir función de validación de estudiante
function validate_estudiante($in) {
    $errors = [];

    if (empty($in['nombre_completo'])) {
        $errors['nombre_completo'] = 'El nombre completo es obligatorio.';
    }
    if (empty($in['documento'])) {
        $errors['documento'] = 'El documento es obligatorio.';
    }
    if (empty($in['programa'])) {
        $errors['programa'] = 'El programa es obligatorio.';
    }
    if (empty($in['celular'])) {
        $errors['celular'] = 'El celular es obligatorio.';
    }
    if (empty($in['email'])) {
        $errors['email'] = 'El email es obligatorio.';
    } elseif (!filter_var($in['email'], FILTER_VALIDATE_EMAIL)) {
        $errors['email'] = 'El email no es válido.';
    }
    if (empty($in['semestre'])) {
        $errors['semestre'] = 'El semestre es obligatorio.';
    }
    if (empty($in['universidad'])) {
        $errors['universidad'] = 'La universidad es obligatoria.';
    }

    return $errors;
}

// Procesar formulario
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $in = $_POST;

    $errors = validate_estudiante($in);
    if (empty($errors) && estudiantes_existe_documento($in['documento'])) {
        $errors['documento'] = 'Ya existe un estudiante con este documento';
    }
    if (!empty($errors)) {
        $_SESSION['old'] = $in;
        $_SESSION['errors'] = $errors;
        flash_set('error', 'Revise los campos del formulario.');
    } else {
        try {
            estudiantes_create($in);
            flash_set('success', 'Registro guardado correctamente.');
        } catch (Throwable $e) {
            flash_set('error', 'Error al guardar: ' . $e->getMessage());
        }
    }
    header('Location: index.php#registro');
    exit;
}
?>

<?php if ($msg = flash_get('error')): ?>
    <div style="color: red;"><?= htmlspecialchars($msg) ?></div>
<?php endif; ?>
<?php if ($msg = flash_get('success')): ?>
    <div style="color: green;"><?= htmlspecialchars($msg) ?></div>
<?php endif; ?>

<!-- Formulario de registro de estudiante -->
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Registro de Estudiante</title>
  <!-- Bootstrap 5 -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light d-flex align-items-center min-vh-100">

  <div class="container">
    <div class="row justify-content-center">
      <div class="col-lg-8 col-md-10">
        <div class="card shadow-sm">
          <div class="card-body">
            <h2 class="text-center mb-4">Registro de Estudiante</h2>
            
            <form action="index.php" method="post" id="registro" class="row g-3">
              
              <div class="col-12">
                <label for="nombre_completo" class="form-label">Nombre completo</label>
                <input type="text" class="form-control" name="nombre_completo" id="nombre_completo" required
                  value="<?= isset($old['nombre_completo']) ? htmlspecialchars($old['nombre_completo']) : '' ?>">
                <?php if (isset($errors['nombre_completo'])): ?>
                  <div class="text-danger small"><?= htmlspecialchars($errors['nombre_completo']) ?></div>
                <?php endif; ?>
              </div>

              <div class="col-md-6">
                <label for="documento" class="form-label">Documento</label>
                <input type="text" class="form-control" name="documento" id="documento" required
                  value="<?= isset($old['documento']) ? htmlspecialchars($old['documento']) : '' ?>">
                <?php if (isset($errors['documento'])): ?>
                  <div class="text-danger small"><?= htmlspecialchars($errors['documento']) ?></div>
                <?php endif; ?>
              </div>

              <div class="col-md-6">
                <label for="programa" class="form-label">Programa</label>
                <input type="text" class="form-control" name="programa" id="programa" required
                  value="<?= isset($old['programa']) ? htmlspecialchars($old['programa']) : '' ?>">
                <?php if (isset($errors['programa'])): ?>
                  <div class="text-danger small"><?= htmlspecialchars($errors['programa']) ?></div>
                <?php endif; ?>
              </div>

              <div class="col-md-6">
                <label for="celular" class="form-label">Celular</label>
                <input type="text" class="form-control" name="celular" id="celular" required
                  value="<?= isset($old['celular']) ? htmlspecialchars($old['celular']) : '' ?>">
                <?php if (isset($errors['celular'])): ?>
                  <div class="text-danger small"><?= htmlspecialchars($errors['celular']) ?></div>
                <?php endif; ?>
              </div>

              <div class="col-md-6">
                <label for="email" class="form-label">Email</label>
                <input type="email" class="form-control" name="email" id="email" required
                  value="<?= isset($old['email']) ? htmlspecialchars($old['email']) : '' ?>">
                <?php if (isset($errors['email'])): ?>
                  <div class="text-danger small"><?= htmlspecialchars($errors['email']) ?></div>
                <?php endif; ?>
              </div>

              <div class="col-md-6">
                <label for="semestre" class="form-label">Semestre</label>
                <input type="text" class="form-control" name="semestre" id="semestre" required
                  value="<?= isset($old['semestre']) ? htmlspecialchars($old['semestre']) : '' ?>">
                <?php if (isset($errors['semestre'])): ?>
                  <div class="text-danger small"><?= htmlspecialchars($errors['semestre']) ?></div>
                <?php endif; ?>
              </div>

              <div class="col-md-6">
                <label for="universidad" class="form-label">Universidad</label>
                <input type="text" class="form-control" name="universidad" id="universidad" required
                  value="<?= isset($old['universidad']) ? htmlspecialchars($old['universidad']) : '' ?>">
                <?php if (isset($errors['universidad'])): ?>
                  <div class="text-danger small"><?= htmlspecialchars($errors['universidad']) ?></div>
                <?php endif; ?>
              </div>

              <div class="col-12">
                <button type="submit" class="btn btn-primary w-100">Registrar</button>
              </div>
            </form>

          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Bootstrap JS -->
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
