# Testdocumentation 1

[hier](https://CGS-TGiesecke.github.io/Testdocumentation/)


Hier ist eine kurze Prüfung/Überblick über den gezeigten GitHub Actions-Workflow:

Ziel: 		Markdown-Dateien in Docs-Pfad werden zu einem einzelnen PDF kombiniert (via Pandoc) und als Artefakt hochgeladen.
Wichtige Punkte:
Trigger: 	push auf main sowie workflow_dispatch (manuelles Ausführen).
			Job md2pdf läuft auf ubuntu-latest.
Schritte enthalten:
	Checkout des Repos (uses: actions/checkout@v4).
	Installation von Pandoc und LaTeX (texlive-full).
	Erstellen eines Build-Verzeichnisses, Generieren der PDF aus docs/.md mit pandoc docs/.md -s -o build/docs.pdf --pdf-engine=pdflatex.
	Upload des erzeugten PDFs als Artefakt docs-pdf.

