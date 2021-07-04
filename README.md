# Veranstaltung Game Engineering

## Dozent: Prof. Dr. Frank Dopatka

[zurück zur Hauptseite…](https://informatik-mannheim.github.io/iExpo-Sommer-2021/)

### Das Projekt

Dieses Semester wurde eine real-time Handelssimulationsplatform entwickelt, die einem erlaubt Dogecoin zu handeln.

### Das Team

* Athanasios Aritzis

## Handelssimulationsplatform TradeX

Das Spiel wurde als Webanwendung entwickelt.

### Backend

Als Backend Server wurde Go verwendet. Es besteht aus zwei Teilen:

1. Exchange:

    * Konsumiert als Goroutine per Websocket API DOGE/USD Live Trades und den Order Book von dem Marktplatz "Binance". Der Order Book wurde als zwei RB-Tree implementiert (für Buy und Sell), sodass die Orders sortiert werden und jede Operation die Zeitkomplexität O(log n) hat. Live Trades werden dem Websocket Controller gesendet.
    * Bearbeitet die Orders und Trades die es vom Controller bekommt, bei Orders handelt es sich hier um die Orders der Spieler, diese werden wenn möglich mit den Orders vom Live-Markt gepaart, und wenn nötig in einem Spieler Order Book gespeichert (zwei RB-Trees). Trades werden dann mit diesen gepaart.
  
2.	Websocket Controller:
    * Steuert alle Websocket Verbindungen. 
    * Stellt User, Chart und Highscore Daten dem Frontend zur Verfügung.
    * Nimmt Handelsanfragen entgegen, sendet sie dem Exchange und speichert diese in die Datenbank
    * Erhällt Live Trades vom Exchange, diese werden an alle aktiven Verbindungen gebroadcastet, wieder dem Exchange übermittelt und in die Datenbank gespeichert.
    * Sollten Trades stattgefunden haben, werden sie dem jeweiligem User gesendet.
    
    
### Frontend
    
Das Frontend ist eine Angular Webanwendung die dem Nutzer die Funktionalitäten anbietet, es stellt eine Websocket Verbindung zum Backend Server her um die Chart, und Highscore Daten abzufragen, Live Trades und Orderbook zu konsumieren, und auch Kauf- und Verkaufsanfragen zu tätigen.
Es wird eine Charting Library von Trading View verwendet.

### Datenbanken

1. Es wird eine reguläre SQL Datenbank PostgreSQL für alle Nutzerdaten, Orders, Trades und Highscores eingesetzt.
2. Eine Timeseries Datenbank InfuxDB wird eingesetzt um alle Live Trades eine bestimmte Zeit aufzubewahren, und diese jede Minute in OHLC ( Open, High, Low, Close) Datensätze zu aggregieren.
