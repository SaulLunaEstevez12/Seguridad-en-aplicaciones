# Seguridad-en-aplicaciones, correccion del codigo
Prevención de Automatización de Eventos
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
