# LinuxAndMore Website
---

## Hallo und willkommen auf meiner Website

In Übereinstimmung mit dem YouTube-Kanal findest du hier Informationen zu technischen Themen, wie z.B. Linux, Server und Hardware, die mich interessieren. Die Idee ist, einen Leitfaden für diejenigen bereitzustellen, die umfassendere Informationen und Lösungen wünschen. Im Prinzip gebe ich meine Erfahrungen und Persönliche Entwicklung in besagten Punkten an euch weiter. Ich würde mich freuen, wenn Ihr mich dabei begleitet.
Was mache ich?

## Fragen und Antworten

Solltet ihr fragen oder Probleme haben erstellt bitte auf der [GitHub-Seite](https://github.com/linuxandmore/linuxandmore.github.io) dieses Projektes ein "issui". Selbverständlich könnt ihr bei inhaltlichen Problemen auch den Inhalt bearbeiten und einen "Pull Request" bzw. "Merge Request" erstellen. Nach entsprechender Validierung, durch mich, wird dann eure Änderung durchgereicht bzw. das Problem genauer betrachtet oder auch euch spezifisch geholfen. 

## Das bin Ich

Hallo, ich bin Nick aus Brandenburg, jahrelanger begeisterter Linuxnutzer und inzwischen wohl Server Junkie. Mit meinen Videos und blogbeitragen versuche ich meine Erfahrungen und Ansichtspunkte mit der Welt zu teilen, um einerseits zur Disskussion anzuregen und vielleicht auch den ein oder anderen zu faszinieren.

## Aktive Projekte

- [Photoboth](https://github.com/LinuxAndMoreYT/Photoboth)
- [Torben-Trinkt](https://github.com/LinuxAndMoreYT/Torben-Trinkt)
- [PXE](https://github.com/LinuxAndMoreYT/PXE)

## Inhalt dieser Seite

{% for post in site.posts %}
  <code>
  <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time>
    <h2>
      <a href="{{ post.url }}">
        {{ post.title }}
      </a>
    </h2>
  </code>
{% endfor %}
