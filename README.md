# Seguridad-en-aplicaciones, correccion del codigo
## Primera vulnerabilidad, Prevención de Automatización de Eventos
Corrección: Se deshabilita temporalmente el botón para evitar múltiples clics.

Antes (Vulnerable a spam de clics)

```dart
ElevatedButton(
  onPressed: () {
    setState(() {
      comments.add(_commentController.text);
      _commentController.clear();
    });
  },
  child: Text('Publicar'),
)
```

## Problema:

No hay validación de frecuencia de clics.
Un script podría hacer clic rápidamente y generar spam.
Después (Código corregido)

```dart
bool _isPublishing = false;

ElevatedButton(
  onPressed: _isPublishing ? null : () async {
    setState(() => _isPublishing = true);
    await Future.delayed(Duration(seconds: 2)); // Simula validación en servidor
    if (_commentController.text.isNotEmpty) {
      setState(() {
        comments.add(_commentController.text);
        _commentController.clear();
      });
    }
    setState(() => _isPublishing = false);
  },
  child: Text('Publicar'),
)
```
## Mejoras:

Botón se deshabilita tras un clic y se reactiva después de 2 segundos.
Evita spam y automatización de interacciones no deseadas.


## Segunda vulnerabilidad, Seguridad en el Inicio de Sesión
Corrección: Se eliminó la validación insegura de credenciales y se reemplazó por Firebase Authentication.

Antes (Código inseguro)

```dart
class LoginScreen extends StatelessWidget {
  final TextEditingController _usernameController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Card(
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(15),
            ),
            elevation: 5,
            child: Padding(
              padding: const EdgeInsets.all(20.0),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  Text('Login', style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold)),
                  SizedBox(height: 20),
                  TextField(
                    controller: _usernameController,
                    decoration: InputDecoration(labelText: 'Username', border: OutlineInputBorder()),
                  ),
                  SizedBox(height: 10),
                  TextField(
                    controller: _passwordController,
                    decoration: InputDecoration(labelText: 'Password', border: OutlineInputBorder()),
                    obscureText: true,
                  ),
                  SizedBox(height: 20),
                  ElevatedButton(
                    style: ElevatedButton.styleFrom(
                      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(10)),
                      padding: EdgeInsets.symmetric(horizontal: 40, vertical: 15),
                    ),
                    onPressed: () {
                      String username = _usernameController.text;
                      String password = _passwordController.text;
                      if (username == 'admin' && password == 'password') {
                        Navigator.push(
                          context,
                          MaterialPageRoute(builder: (context) => HomeScreen(username: username)),
                        );
                      }
                    },
                    child: Text('Login'),
                  ),
                  TextButton(
                    onPressed: () {
                      Navigator.push(
                        context,
                        MaterialPageRoute(builder: (context) => RegisterScreen()),
                      );
                    },
                    child: Text('Register'),
                  )
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

## Problema:

Las credenciales están en el código (Un atacante podría encontrarlas fácilmente).
No hay encriptación ni autenticación real.
Vulnerable a inyección SQL si se usa una base de datos directa.
Después (Código corregido)
```dart
import 'package:firebase_auth/firebase_auth.dart';


class LoginScreen extends StatefulWidget {
  @override
  _LoginScreenState createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  bool _isLoading = false; // Para evitar múltiples intentos seguidos

  Future<void> _login() async {
    setState(() => _isLoading = true);
    try {
      await FirebaseAuth.instance.signInWithEmailAndPassword(
        email: _emailController.text.trim(),
        password: _passwordController.text.trim(),
      );
      Navigator.pushReplacement(
        context,
        MaterialPageRoute(builder: (context) => HomeScreen(username: _emailController.text)),
      );
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text("Error al iniciar sesión. Verifica tus datos")),
      );
    }
    setState(() => _isLoading = false);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Card(
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(15),
            ),
            elevation: 5,
            child: Padding(
              padding: const EdgeInsets.all(20.0),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  Text('Login', style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold)),
                  SizedBox(height: 20),
                  TextField(
                    controller: _emailController,
                    decoration: InputDecoration(labelText: 'Correo Electrónico', border: OutlineInputBorder()),
                    keyboardType: TextInputType.emailAddress,
                  ),
                  SizedBox(height: 10),
                  TextField(
                    controller: _passwordController,
                    decoration: InputDecoration(labelText: 'Contraseña', border: OutlineInputBorder()),
                    obscureText: true,
                  ),
                  SizedBox(height: 20),
                  ElevatedButton(
                    style: ElevatedButton.styleFrom(
                      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(10)),
                      padding: EdgeInsets.symmetric(horizontal: 40, vertical: 15),
                    ),
                    onPressed: _isLoading ? null : _login, // Llama a la función _login
                    child: _isLoading ? CircularProgressIndicator() : Text('Login'),
                  ),
                  TextButton(
                    onPressed: () {
                      Navigator.push(
                        context,
                        MaterialPageRoute(builder: (context) => RegisterScreen()),
                      );
                    },
                    child: Text('Register'),
                  )
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

## Mejoras:
La función _login ahora está dentro de _LoginScreenState y usa Firebase para autenticar usuarios.
Se agregaron controles para evitar múltiples inicios de sesión seguidos, usando _isLoading.
El botón de "Login" ahora usa _login correctamente.
Se cambió controller: _usernameController por controller: _emailController, ya que Firebase usa correos electrónicos para autenticar usuarios.
El botón ahora desactiva el login si _isLoading es true, evitando clics múltiples.
Al iniciar sesión con éxito, se navega a HomeScreen usando Navigator.pushReplacement(), para que el usuario no pueda volver atrás con el botón de retroceso.
