# Catalogo de Músicas

Este é um aplicativo simples para catalogar músicas utilizando Flutter.

## Instruções de Configuração e Execução

### Pré-requisitos

- Flutter SDK instalado
- Dispositivo Android/iOS ou emulador configurado

### Configuração

1. Clone este repositório:

   ```bash
   git clone https://github.com/seu-usuario/nome-do-repositorio.git
   cd nome-do-repositorio
2. Executa ai
flutter run
import 'package:catalogomusica/controllers/musica_controller.dart';
import 'package:catalogomusica/models/musica.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const CatalogoMusicas());
}

// Classe principal do aplicativo
class CatalogoMusicas extends StatelessWidget {
  const CatalogoMusicas({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Catalogo de Musicas',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const CatalogoMusicaPage(),
    );
  }
}

// Página principal que carrega e exibe a lista de músicas
class CatalogoMusicaPage extends StatefulWidget {
  const CatalogoMusicaPage({super.key});

  @override
  State<CatalogoMusicaPage> createState() => _CatalogoMusicaPage();
}

class _CatalogoMusicaPage extends State<CatalogoMusicaPage> {
  final MusicaController musicaController = MusicaController();
  late Future<List<Musica>> _catalogoMusica;
  bool _mostraSomenteNao = false;

  @override
  void initState() {
    super.initState();
    _catalogoMusica = musicaController.getMusicas();
  }

  // Função para filtrar músicas
  void _chamaMusica() {
    setState(() {
      _catalogoMusica = musicaController.getMusicas(comprado: _mostraSomenteNao ? true : null);
    });
  }

  // Função para atualizar a lista de músicas
  void _refreshList() {
    setState(() {
      _catalogoMusica = musicaController.getMusicas();
    });
  }

  // Função para exibir formulário de adição/edição de música
  void _showForm([Musica? musica]) {
    final formKey = GlobalKey<FormState>();
    String titulo = musica?.titulo ?? '';
    String artista = musica?.artista ?? '';
    String album = musica?.album ?? '';
    String genero = musica?.genero ?? '';

    showDialog(
      context: context,
      builder: (context) {
        return AlertDialog(
          title: Text(musica == null ? 'Adicionar Música' : 'Editar Música'),
          content: Form(
            key: formKey,
            child: Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                TextFormField(
                  initialValue: titulo,
                  decoration: const InputDecoration(labelText: "Título"),
                  onSaved: (value) => titulo = value!,
                ),
                TextFormField(
                  initialValue: artista,
                  decoration: const InputDecoration(labelText: "Artista"),
                  onSaved: (value) => artista = value!,
                ),
                TextFormField(
                  initialValue: album,
                  decoration: const InputDecoration(labelText: "Álbum"),
                  onSaved: (value) => album = value!,
                ),
                TextFormField(
                  initialValue: genero,
                  decoration: const InputDecoration(labelText: "Gênero"),
                  onSaved: (value) => genero = value!,
                ),
              ],
            ),
          ),
          actions: [
            TextButton(
              onPressed: () {
                Navigator.of(context).pop();
              },
              child: const Text('Cancelar'),
            ),
            TextButton(
              onPressed: () {
                if (formKey.currentState!.validate()) {
                  formKey.currentState!.save();
                  final novaMusica = Musica(
                    id: musica?.id,
                    titulo: titulo,
                    artista: artista,
                    album: album,
                    genero: genero,
                  );
                  if (musica == null) {
                    musicaController.addMusica(novaMusica).then((_) => _refreshList());
                  } else {
                    musicaController.updateMusica(novaMusica).then((_) => _refreshList());
                  }
                  Navigator.of(context).pop();
                }
              },
              child: const Text('Salvar'),
            ),
          ],
        );
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Catálogo de Músicas'),
        actions: [
          // Switch para filtrar músicas compradas/não compradas
          Switch(
            value: _mostraSomenteNao,
            onChanged: (value) {
              setState(() {
                _mostraSomenteNao = value;
                _chamaMusica();
              });
            },
            activeColor: Colors.green,
            inactiveThumbColor: Colors.red,
          ),
        ],
      ),
      // Corpo da página que exibe a lista de músicas
      body: FutureBuilder<List<Musica>>(
        future: _catalogoMusica,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(
              child: CircularProgressIndicator(),
            );
          } else if (snapshot.hasError) {
            return Center(
              child: Text('Erro: ${snapshot.error}'),
            );
          } else if (!snapshot.hasData || snapshot.data!.isEmpty) {
            return const Center(
              child: Text('Sem músicas!'),
            );
          }
          final musicas = snapshot.data!;
          return ListView.builder(
            itemCount: musicas.length,
            itemBuilder: (context, index) {
              final musica = musicas[index];
              return ListTile(
                title: Text(musica.titulo),
                // Ao pressionar e ao tocar, exibir o formulário de edição
                onLongPress: () => _showForm(musica),
                onTap: () => _showForm(musica),
              );
            },
          );
        },
      ),
      // Botão flutuante para adicionar nova música
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showForm(),
        child: const Icon(Icons.add),
      ),
    );
  }
}
