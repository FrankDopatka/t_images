# Veranstaltung Game Engineering

## Dozent: Prof. Dr. Frank Dopatka

[zurück zur Hauptseite…](https://informatik-mannheim.github.io/iExpo-Sommer-2021/)

### Das Projekt

Dieses Semester wurde eine real-time Handelssimulationsplatform entwickelt, die einem erlaubt Dogecoin zu handeln. Die Top 10 der Nutzer die innerhalb 30 Minuten den größten Gewinn erziehlen, erscheinen für alle anderen Nutzer im Highscore.

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
![ui](https://user-images.githubusercontent.com/12783903/124397915-419f2d00-dd13-11eb-909a-d5fc0bf4e68b.png)

Das Frontend ist eine Angular Webanwendung die dem Nutzer die Funktionalitäten anbietet, es stellt eine Websocket Verbindung zum Backend Server her um die Chart und Highscore Daten abzufragen, Live Trades und Orderbook zu konsumieren, und auch Kauf- und Verkaufsanfragen zu tätigen.
Es wird eine Charting Library von Trading View verwendet.

#### Chart
![ohlc](https://user-images.githubusercontent.com/12783903/124398572-a314cb00-dd16-11eb-9798-2832ff571478.png)

Der Chart beinhaltet die Kursdaten der letzten 24 Stunden. Jeder Balken / jede Kerze ergibt sich aus den Preis des ersten und letzten Trades, sowie Höchst- und Tiefpreis innerhalb einer Minute. Ist der Preis des letzten Trades größer als des Ersten, dann ist die Kerze grün, sonst rot.

#### Timer

Oben Links gibt es einen "Start Timer" Button, diesen muss man anklicken damit man Orders abgeben kann, es erscheint ein Countdown, ist dieser abgelaufen ist das Spiel zu Ende, es werden alle offenen Orders abgebrochen, Positionen geschlossen und die Wertung wird im Highscore eingetragen.

#### Trading Terminal
![terminal](https://user-images.githubusercontent.com/12783903/124398237-ee2dde80-dd14-11eb-8940-922f72ae8108.png)

Hier hat man die Option Orders abzugeben, wieviele DOGE man kaufen/verakaufen möchte und ob zu einem festem Preis, im "Total" Feld wird die Gesammtsumme errechnet die man für den Verkauf/Kauf benötigt. Lässt man den Limit weg, wird zum Marktpreis gekauft/verkauft.

Außerdem hat man noch die Möglichkeit seine Order ablaufen zu lassen wenn diese innerhalb einer bestimmten Zeitspanne nicht gefüllt wurde, indem man den IOC (Immediate or Cancel) Button aktiviert. Lässt man die Zeitspanne weg, wird die Order sofort storniert. Dies kann zu nutzen sein wenn man zum Marktpreis bis zu einem bestimmten Preis verkaufen / kaufen möchte.

#### Orders / Trades
![ot](https://user-images.githubusercontent.com/12783903/124398268-1ddce680-dd15-11eb-9d47-163b9583af9c.png)

Alle Orders und Trades werden in einer Tabelle bereitgestellt

#### Position
![pos](https://user-images.githubusercontent.com/12783903/124398483-326dae80-dd16-11eb-8bee-b59f88ef6a83.png)

Eine Position kann entweder "Long" oder "Short" sein. Wenn man kauft geht man eine Long Position ein, um diese zu schließen muss man verkaufen, verkauft man zu einem höherem Preis profitiert man.Wenn man verkauft geht man eine Short Position ein, effektiv leiht man sich den Wert den man verkauft. Wenn man also zu einem niedrigerem Preis wieder kauft, dann gibt man effektiv den geliehten Betrag zurück und behällt die Differenz.

#### Score
![high](https://user-images.githubusercontent.com/12783903/124398502-487b6f00-dd16-11eb-83df-2b164c4bceb8.png)

Der Score entspricht den prozentualen Gewinn den man erbracht hat, befindet man sich nicht in den Top 10, wird unser bester Userscore und Rang zuletzt angezeigt.


### Datenbanken

1. Es wird eine reguläre SQL Datenbank PostgreSQL für alle Nutzerdaten, Orders, Trades und Highscores eingesetzt.
2. Eine Timeseries Datenbank InfuxDB wird eingesetzt um alle Live Trades eine bestimmte Zeit aufzubewahren, und diese jede Minute in OHLC ( Open, High, Low, Close) Datensätze zu aggregieren.
