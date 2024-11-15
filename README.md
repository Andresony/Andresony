flutter create clima_trafico_bogota
cd clima_trafico_bogota
dependencies:
  flutter:
    sdk: flutter
  http: ^0.13.4
  google_maps_flutter: ^2.1.1
import 'package:flutter/material.dart';
import 'screens/home_screen.dart';

void main() {
  runApp(ClimaTraficoBogotaApp());
}

class ClimaTraficoBogotaApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Clima y Tráfico en Bogotá',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: HomeScreen(),
    );
  }
}
import 'package:flutter/material.dart';

class HomeScreen extends StatelessWidget {
  final TextEditingController originController = TextEditingController();
  final TextEditingController destinationController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Clima y Tráfico en Bogotá'),
      ),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Column(
          children: [
            TextField(
              controller: originController,
              decoration: InputDecoration(
                labelText: 'Origen',
              ),
            ),
            SizedBox(height: 10),
            TextField(
              controller: destinationController,
              decoration: InputDecoration(
                labelText: 'Destino',
              ),
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () {
                // Navegar a la pantalla de tráfico y clima
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => TrafficWeatherScreen(
                    origin: originController.text,
                    destination: destinationController.text,
                  )),
                );
              },
              child: Text('Ver Clima y Tráfico en Ruta'),
            ),
          ],
        ),
      ),
    );
  }
}import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';

class TrafficWeatherScreen extends StatefulWidget {
  final String origin;
  final String destination;

  TrafficWeatherScreen({required this.origin, required this.destination});

  @override
  _TrafficWeatherScreenState createState() => _TrafficWeatherScreenState();
}

class _TrafficWeatherScreenState extends State<TrafficWeatherScreen> {
  String weather = 'Cargando...';
  String traffic = 'Cargando...';

  @override
  void initState() {
    super.initState();
    fetchWeather();
    fetchTraffic();
  }

  Future<void> fetchWeather() async {
    // Ejemplo de API de clima: OpenWeatherMap
    String apiKey = 'TU_API_KEY';
    String url = 'https://api.openweathermap.org/data/2.5/weather?q=Bogota&appid=$apiKey';
    final response = await http.get(Uri.parse(url));
    if (response.statusCode == 200) {
      var data = json.decode(response.body);
      setState(() {
        weather = 'Clima: ${data['weather'][0]['description']}, Temp: ${(data['main']['temp'] - 273.15).toStringAsFixed(1)}°C';
      });
    } else {
      setState(() {
        weather = 'Error al obtener clima';
      });
    }
  }

  Future<void> fetchTraffic() async {
    // Ejemplo de API de tráfico: Google Maps Directions API
    String apiKey = 'TU_API_KEY';
    String url = 'https://maps.googleapis.com/maps/api/directions/json?origin=${widget.origin}&destination=${widget.destination}&key=$apiKey';
    final response = await http.get(Uri.parse(url));
    if (response.statusCode == 200) {
      var data = json.decode(response.body);
      setState(() {
        traffic = 'Duración estimada: ${data['routes'][0]['legs'][0]['duration']['text']}';
      });
    } else {
      setState(() {
        traffic = 'Error al obtener tráfico';
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Clima y Tráfico'),
      ),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('Origen: ${widget.origin}'),
            Text('Destino: ${widget.destination}'),
            SizedBox(height: 20),
            Text(weather),
            SizedBox(height: 20),
            Text(traffic),
          ],
        ),
      ),
    );
  }
}
flutter run

