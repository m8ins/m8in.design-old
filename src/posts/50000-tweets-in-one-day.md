---
author: Martin Schmitz
date: '2020-05-30'
image: '/images/2020/server.jpg'
tags:
  - master
  - twitter
  - coding
title: Wie und warum ich 50.000 Tweets in einem Tag gesammelt habe
summary: 'Kurzer Erfahrungsbericht aus den Anfängen meiner Master-Arbeit'
---

# Wie und warum ich 50.000 Tweets in einem Tag gesammelt habe

Die letzten Klausuren sind bestanden, die letzten Seminare hinter mich gebracht und vor meinem Abschluss steht jetzt nur noch meine Masterarbeit. In den kommenden Monaten werde ich mich damit befassen, wie man Laien dabei unterstützen kann, Twitter-Diskussionen besser einordnen zu können. Dabei konzentriere ich mich zu Beginn zunächst einmal darauf, große Datenmengen in Grafiken darzustellen. Dadurch kann man dann etwa erkennen, ob negative Tweets von vielen verschiedenen Leuten kommen, oder ob ein paar “Superspreader” einfach nur schlechte Laune auf Twitter verbreiten wollen.

Doch bevor ich damit anfangen kann, mir Gedanken über coole Darstellungsformen zu machen, brauche ich natürlich erst einmal: Daten, Daten und noch mehr Daten. Zum Glück bietet Twitter eine mächtige, wenn auch nicht gerade angenehm zu nutzende, API[^1] an, die mir das Sammeln deutlich erleichtert hat. Für den Zugriff auf die API habe ich Python und [Tweepy](https://github.com/tweepy/tweepy) benutzt.

## Was kann die Twitter-API, und was fange ich damit an?

Twitter lässt einen mit der API auf verschiedene Arten nach Tweets suchen. Man kann zum Beispiel das Twitter-Archiv nach Nachrichten mit einem bestimmten Schlagwort durchsuchen. Dazu formuliert man eine Suchanfrage, die inhaltlich etwa “Gib mir bis zu 20.000 Tweets aus den vergangenen 7 Tagen, die mit den Keywords “Corona” oder “#Corona” abgeschickt wurden”. Allerdings werden die Tweets, die man auf so eine Suchanfrage zurückbekommt, zufällig ausgewählt. Eine andere Möglichkeit ist das _Streaming_, das ich auch für meine Master-Arbeit benutze. Beim Streaming bittet man Twitter darum, alle Tweets weiterzuleiten, die zu bestimmten Keywords abgeschickt werden. Meine Suchanfrage sieht etwa so aus:

```python
stream.filter(track=['corona', 'distancing','covid-19', 'covidioten'], languages=['de'])
```

`stream.filter` ist die Funktion, die meine Streaming-Verbindung mit Twitter aufbaut. `track` ist eine Liste von Schlagwörtern, nach denen ich filtern möchte – in meiner Suchanfrage also “Corona”, “Distancing” und so weiter. Wenn einer oder mehrere dieser Begriffe in einem Tweet auftauchen, wird er an mein Python-Skript weitergeleitet. Mit `languages=['de']` filtere ich zusätzlich noch nach deutschsprachigen Tweets. Hier könnte ich theoretisch auch wieder eine Liste von verschiedenen Sprachen mitgeben, für meine Arbeit konzentriere ich mich aber auf deutschsprachige Tweets.

## Wie werden die ganzen Daten gespeichert?

Die Tweets werden [in BigQuery gespeichert](https://cloud.google.com/bigquery/), der SQL-Datenbank von Google. Dafür habe ich sogar eine waschechte Big Data-Pipeline aufgebaut! Hintergrund des Ganzen ist, dass ich ziemlich viele Daten sammele – knapp 50.000 Twitter-Nachrichten am Tag, also etwa anderthalb Tweets pro Sekunde. Und die müssen alle nahezu in Echtzeit verarbeitet und gespeichert werden. Meine Pipeline sieht so aus:

1. Ein Python-Skript empfängt den Twitter-Stream und bereitet die von Twitter gesendeten Daten auf
2. Anschließend veröffentlicht das Skript die aufbereiteten Tweets in ein sogenanntes [PubSub-Thema](https://cloud.google.com/pubsub/). Das kann man sich wie eine Radio-Antenne vorstellen: Die Daten werden erst einmal rausgeschickt, egal ob jemand zuhört oder nicht.
3. Ein zweites Python-Skript läuft periodisch und abonniert das PubSub-Thema. Wenn neue Nachrichten da sind, werden sie entschlüsselt und in die Datenbank geschrieben
4. Am Ende der Pipeline steht dann BigQuery als Datenbank. Hier kann ich dann Abfragen auf meinen Daten ausführen – etwa, wie viele Tweets an einem bestimmten Tag abgeschickt wurden oder wie viele Follower die Twitteruser:innen durchschnittlich hatten.

Die Skripte laufen in einem Docker-Container über Google AppEngine, weil zumindest das Skript mit dem Twitter-Stream Tag und Nacht durchlaufen muss.
![Ein dunkler Raum mit Servern an den Wänden](/images/2020/server.jpg 'Die Daten werden von einem Server auf einen anderen Server geschoben um auf einen dritten Server geschoben zu werden. Cloud Computing at its best.')

## Welche Daten sammle ich genau?

Wie ich oben schon geschrieben habe, stellt die Twitter-API einem für jeden Tweet einen riesigen Haufen an Daten zur Verfügung. Neben dem eigentlichen Tweet-Text und dem Autor zum Beispiel auch die Hintergrundfarbe des Autorenprofils, der Längen- und Breitengrad, auf dem der Tweet abgesetzt wurde, die Größe des Thumbnail-Bilds in Pixeln undundund…

Jeder Tweet wird als JSON-Objekt an mein Python-Skript geschickt. Theoretisch kann BigQuery mit JSON-Objekten umgehen, sodass ich all diese Daten einfach hätte “weiterschieben” können. Dieser [Beitrag auf Signal vs. Noise](https://m.signalvnoise.com/marking-the-end-of-pixel-trackers-in-basecamp-emails/) hat mich aber zum Nachdenken angeregt:

> The tech industry has been so used to capturing whatever data it could for so long that it has almost forgotten to ask whether it should. But that question is finally being asked. And the answer is obvious: This gluttonous collection of data must stop.

Anstatt also alle Daten zu sammeln, die ich _irgendwie_ in die Finger bekomme, wollte ich mich lieber bewusst für und vor allem gegen bestimmte Aspekte entscheiden. Schlussendlich werden jetzt folgende Daten auch in der Datenbank gespeichert:

- Der Tweet-Text
- Der Display-Name des oder der Verfasser:in
- Die Liste der Hashtags, die im Tweet genutzt wurden
- die ID des Tweets – damit kann ich nachher Tweets auch zuverlässig auseinander halten
- Die Anzahl der Freunde und Follower des Verfassers oder der Verfasserin. Das könnte für Auswertungen interessant sein – etwa um zu schauen, ob sich das Verhalten auf Twitter zwischen großen und kleinen Accounts unterschiedet
- Das Land, aus dem der Tweet abgesetzt wurde
- Ob der Account verifiziert ist oder nicht
- Der Zeitpunkt, zu dem der Tweet an den Streamer geschickt wurde
- Die Quelle des Tweets – wurde die Nachricht vom Handy oder vom Computer aus geschickt?
- Das Sentiment des Tweets, also eine Zahl zwischen -1 und 1, die die grundlegende “Stimmung” im Tweet beschreibt. Stark vereinfacht gesagt heißt -1, dass der Tweet eher grummelig ist; +1, dass wohl jemand mit guter Laune an der Tastatur saß

Das Sentiment habe ich zuerst über die Google Natural Language API berechnen lassen. Das hat aber leider 50 Euro am Tag gekostet (hupps), deshalb benutze ich jetzt Textblob zur Berechnung des Sentiments[^2]. Abgesehen vom teuren Sentiment-Ausrutscher bin ich bisher aber sehr zufrieden mit der Pipeline.

Innerhalb der vergangenen 7 Tage konnte ich schon **mehr als 338.904 deutschsprachige Tweets** mit den oben genannten Keywords sammeln – eine Zahl, mit der ich zu Beginn nicht gerechnet hätte. Zum Glück habe ich mich dazu entschieden, auf eine SQL-Datenbank zu setzen und die Pipeline mithilfe des Pub/Sub-Patterns stabil zu bauen. Bei diesen Datenmengen zahlt sich die extra Arbeit auf jeden Fall aus.

## Gedanken zum Schluss

Zum Schluss vielleicht noch ein paar Gedanken zur Twitter-API: Es macht leider nicht wirklich Spaß, damit zu arbeiten.
Das fängt schon mit der Dokumentation der API an. Ich kann beim besten Willen nicht finden, wie das vollständige JSON-Statusobjekt aussieht. Meine Lösung war letztendlich, ein Datenobjekt abzurufen, dieses Beispiel-Objekt zu speichern und mein eigenes Beispiel als Referenz zu benutzen. Das Problem dabei war, dass ich mir zufällig einen kurzen Tweet ausgesucht habe, der noch in Twitters altes 140-Zeichen-Format passt. Das Daten-Objekt eines 280-Zeichen-Tweets sieht dagegen anders aus, wodurch ich mir ein zweites Beispiel-Objekt raussuchen und speichern musste; das fiel mir aber erst auf, nachdem mein Skript in einen Fehler gelaufen ist und der Stream abbrach.

Beim Arbeiten mit der API hatte ich das Gefühl, dass an keiner Stelle für mich mitgedacht wird. Bei einem 280-Zeichen-Tweet muss ich beispielsweise das Attribut `fulltext` auslesen, um an den gesamten Text zu kommen (und nicht an eine abgekürzte 140-Zeichen-Version). Ich kann aber nicht einfach immer den `fulltext` abfragen, da Tweets mit weniger als 141 Zeichen dieses Attribut gar nicht haben. Für diese Tweets muss ich dann also wieder in das `text`-Attribut gucken. An solchen Stellen hätte ich mir gewünscht, dass die API mir ein bisschen Arbeit abnimmt zum Beispiel jedem Tweet ein Attribut `fulltext` gibt, dass mal mehr und mal weniger als 140 Zeichen hat. Dieses _nicht-mitdenken_ zieht sich leider ein bisschen durch die API; bei jeder Benennung, jedem Datenfeld, jedem Attribut musste ich darüber nachdenken wie das heißt. In einen richtigen Flow kam ich da nicht.

Aber jetzt, wo ich mich mit der API einigermaßen einigen konnte und an die Daten komme, kann ich auch endlich mit der "richtigen" Arbeit loslegen – schließlich geht es in meiner Masterarbeit ja gar nicht darum, wie man viele Tweets sammeln kann. Ich bin schon mal gespannt, was ich dabei so rausfinde!

[^1]: Eine API ist eine Schnittstelle, mit der ich meine selbst programmierten Sachen einfach an bestehende Twitter-Funktionalitäten “anstöpseln” kann. Im Gegensatz zu sogenannten _Crawlern_, die sich wie menschliche Nutzer durch Twitter durchklicken würden und so Daten sammeln, bekommt man bei einer API deutlich schnelleren und umfassenderen Zugriff auf die Twitter-Daten.
[^2]: Textblob berechnet das Sentiment auf Grundlage eines Wörterbuchs, in dem den Wörtern Punkte zugewiesen wurden. Um das Sentiment zu berechnen, zählt Textblob die Punkte der einzelnen Wörter zusammen. Bei Tweets mit vielen Abkürzungen und Emojis kommen bei einem Ansatz, der auf einem annotierten Wörterbuch beruht, leider keine sonderlich guten Ergebnisse raus. Daher werde ich mich in den kommenden Tagen damit befassen, das Sentiment von einer speziell trainierten künstlichen Intelligenz auswerten zu lassen, die bei Tweets bessere Ergebnisse erzielt.
