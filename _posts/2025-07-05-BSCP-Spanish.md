---
title: "Mi experiencia con el BSCP: análisis, consejos y trucos"
date: 2025-07-05 14:10:00 +0800
author: pablo
categories: [certifications, BSCP]
tags: [
    BSCP
    Review
    Pentesting
    AppSec
  ]
image:
  path: /assets/img/BSCPExam/BSCPCertificate.png
---

Hola! Si estás leyendo esto, es muy probable que estés interesado en rendir el examen BSCP de PortSwigger. En esta publicación quiero compartirte mi experiencia personal con el examen, cómo me preparé, qué materiales de estudio recomiendo y algunos consejos clave que te pueden ayudar a aprobarlo con éxito

---

## Acerca de mí & **Background**

Desde que descubrí el mundo de la ciberseguridad, durante la época de la pandemia, siempre me sentí atraído por su lado ofensivo, especialmente por temas como el pentesting y el red teaming. Por esa razón, me enfoqué en aprender y fortalecer mis habilidades como pentester, lo que eventualmente me llevó a rendir y obtener certificaciones como la eCCPT y la CAPEN. Sin embargo, sentía que necesitaba reforzar mis conocimientos en hacking web y AppSec, por lo que identifiqué tres certificaciones clave en esa área: la CBBH de Hack The Box, la eWPT de INE y la BSCP de PortSwigger.

En ese punto, debía decidir cuál de las certificaciones se alineaba mejor con mis objetivos. Comencé explorando la CBBH, ya que Hack The Box ofrece a través de su plataforma Academy una suscripción mensual de solo $8 para estudiantes, lo cual me pareció una excelente opción. Aproveché esa oportunidad para completar el path de *Bug Bounty Hunter*, diseñado específicamente por HTB como preparación para su certificación CBBH.

![image.png](/assets/img/BSCPExam/image.png)
_Path Completado CBBH_

Disfruté y aprendí muchísimo realizando el path de la CBBH. El material de Hack The Box es, honestamente, muy bueno: la forma en que explican los conceptos y la calidad de los laboratorios lo hacen muy recomendable. Sin embargo, sentía que aún faltaba mayor profundidad en algunos temas clave, y por eso no terminaba de alinearse con mi objetivo principal: profundizar al máximo en todo lo relacionado con hacking web y AppSec.

Fue entonces cuando finalmente me decidí por la certificación BSCP. Tras leer varias reseñas, la mayoría coincidía en que se trata de una certificación exigente, donde muchos candidatos necesitan más de un intento para aprobarla. Esto deja en claro que debes tener muy bien asimilados los conceptos y contar con una base sólida en hacking web para superarla. Además, su precio accesible de $99 terminó de convencerme.

En cuanto a la eWPT de INE, muchas opiniones mencionaban que su contenido era algo básico, y como mi enfoque era aprender lo más posible sobre seguridad web, decidí dejarla de lado por el momento. Tal vez en un futuro considere la nueva versión, eWPTXv3, que según varios comentarios es mucho más desafiante y profunda.

Perdón por la larga introducción, pero creo que era importante compartir todo este contexto y mis pensamientos antes de entrar de lleno en la review de la certificación.

## **¿Qué es el BSCP?**

La **BSCP** es una certificación oficial otorgada por los creadores de Burp Suite, orientada a profesionales en seguridad web. Demuestra un dominio profundo de vulnerabilidades web, la mentalidad adecuada para explotarlas de forma ética, y habilidades avanzadas en el uso de Burp Suite para pruebas de penetración.

📚 **La certificación trata en profundidad temas como:**

- SQL Injection
- Cross-Site Scripting (XSS)
- Vulnerabilidades de control de acceso (horizontal y vertical)
- Server-Side Request Forgery (SSRF)
- Vulnerabilidades en lógica de negocio
- Explotación de web shells
- Uso avanzado de herramientas como Repeater, Intruder, Decoder, Comparer y extensiones

...y muchos más.

🧪 El examen dura 4 horas, cuesta $99 USD y está dividido en tres niveles:

- **Nivel 1:** Acceder a cuentas de usuario estándar
- **Nivel 2:** Escalar privilegios para obtener acceso administrativo
- **Nivel 3:** Explotar la interfaz administrativa para leer un archivo local

Es importante destacar que para rendir el examen necesitas contar con una versión activa de Burp Suite Professional, ya que al finalizarlo exitosamente deberás subir tu proyecto de Burp a la plataforma del examen. PortSwigger ofrece la posibilidad de obtener una prueba gratuita de 30 días si dispones de un correo educativo, que fue la opción que utilicé en mi caso. Así que, si no tienes una licencia activa, esta es una alternativa totalmente válida.

Otro aspecto importante a tener en cuenta durante el examen es que se trata de una prueba **proctorizada**. Antes de comenzar, la plataforma te solicitará subir una imagen de tu documento de identidad, activar la cámara y compartir la pantalla. Aunque en algunos blogs mencionan que, una vez completado el proceso de identificación, es posible cerrar la pestaña del proctoring, yo preferí dejarla abierta para evitar posibles inconvenientes.

Te recuerdo que el examen es **open-book**, por lo que podrás utilizar tus apuntes y herramientas automatizadas, como **sqlmap**, si lo consideras necesario.

## Web Security Academy

La Web Security Academy de PortSwigger es, sin duda, el mejor lugar para prepararte para el examen, y esto se debe a varias razones. En primer lugar, el entorno del examen es prácticamente idéntico al de los laboratorios de la academia, por lo que si ya estás familiarizado con ellos, te sentirás cómodo desde el primer momento. En segundo lugar, es posible que algunas vulnerabilidades del examen te resulten familiares si previamente has trabajado en laboratorios similares. Eso sí, incluso si se repiten, probablemente debas adaptar el enfoque o modificar el exploit según el contexto del entorno. En tercer lugar, todas las vulnerabilidades que pueden aparecer en el examen están cubiertas en la academia, lo que hace que su contenido sea más que suficiente para aprobar con éxito.

PortSwigger no especifica exactamente qué módulos debes completar antes de rendir el examen; simplemente recomienda haber realizado todos los laboratorios de nivel “Apprentice” y “Practitioner”. En mi caso, además, completé algunos laboratorios del nivel “Expert” para reforzar aún más mi preparación. Si te sirve como referencia, este era mi progreso en la academy al momento de presentarme al examen:

![image.png](/assets/img/BSCPExam/image%201.png)

PortSwigger recomienda poner especial énfasis en ciertos laboratorios, por lo que te sugiero revisarlos con mucha atención y tenerlos muy presentes al momento de rendir el examen.

![image.png](/assets/img/BSCPExam/image%202.png)

## Sobre el examen

Como había mencionado previamente el examen dura 4 horas, cuesta $99 USD y está dividido en dos aplicaciones web cada una con tres niveles:

- **Nivel 1:** Acceder a cuentas de usuario estándar
- **Nivel 2:** Escalar privilegios para obtener acceso administrativo
- **Nivel 3:** Explotar la interfaz administrativa para leer un archivo local

Para rendir el examen no existen requisitos obligatorios, pero es fundamental estar bien preparado. Por eso, te recomiendo leer cuidadosamente las siguientes guías, donde se aclaran en detalle todos los aspectos importantes del examen:

- [https://portswigger.net/web-security/certification/practice-exam](https://portswigger.net/web-security/certification/practice-exam)
- [https://portswigger.net/web-security/certification/how-it-works](https://portswigger.net/web-security/certification/how-it-works)
- [https://portswigger.net/web-security/certification/practice-exam](https://portswigger.net/web-security/certification/practice-exam)
- [https://portswigger.net/web-security/certification/exam-hints-and-guidance](https://portswigger.net/web-security/certification/exam-hints-and-guidance)

Para afrontar el examen de la mejor manera, te recomiendo enfocar tu análisis según el nivel de acceso en el que te encuentres dentro de la aplicación. Por ejemplo, no tendría sentido buscar vulnerabilidades como `OS Command Injection` si aún no has conseguido acceso como usuario administrador.

Para ayudarte con este enfoque, te comparto las siguientes checklists, organizadas por nivel de acceso y centradas en las vulnerabilidades más probables en cada etapa del examen. (En la sección de recursos encontrarás las referencias de donde obtuve estas imágenes).

![image.png](/assets/img/BSCPExam/image%203.png)

![image.png](/assets/img/BSCPExam/image%204.png)

Check list interactiva:

[https://bscp.guide/](https://bscp.guide/)

## ¿Cómo preparse?

Considero que el tiempo de preparación y la forma de afrontarlo puede variar mucho según el background y la experiencia previa que cada persona tenga en hacking web. Por eso, a continuación, quiero compartirte cómo fue mi proceso de preparación, junto con algunas recomendaciones y ajustes que haría si tuviera que rendir el examen nuevamente, ahora que conozco su estructura y nivel de exigencia.

- En primer lugar, te recomiendo completar todos los laboratorios de todos los temas en los niveles “Apprentice” y “Practitioner”.
- Una vez que hayas completado todos los laboratorios, te recomiendo volver a hacerlos, pero esta vez tomando notas detalladas sobre la metodología utilizada en cada uno. Esto te permitirá crear tu propia *cheat sheet* personalizada. Aunque en internet existen muchas *cheat sheets* ya elaboradas, desarrollar la tuya propia te ayudará a interiorizar mejor los conceptos y reforzar lo aprendido.
- En la siguiente imagen te muestro un ejemplo de cómo organizaba mis notas utilizando Notion, en este caso sobre la vulnerabilidad CSRF. Crear tu propia *cheat sheet* no solo te será útil para aprobar el examen, sino también para poder extrapolar todo ese conocimiento y metodología a escenarios reales de pruebas de penetración o programas de bug bounty.


![image.png](/assets/img/BSCPExam/image%205.png)

- Dos de los mejores *cheat sheets* que encontré fueron los siguientes. Te recomiendo compararlos con tu propia *cheat sheet* y añadir todo lo que consideres necesario para completarla.
    - [https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Practitioner-Exam-Study)
    - [https://github.com/DingyShark/BurpSuiteCertifiedPractitioner](https://github.com/DingyShark/BurpSuiteCertifiedPractitioner)
- Cuando trabajes en los laboratorios, no te limites únicamente a cumplir el objetivo que se te plantea. Intenta siempre ir un paso más allá y elevar la criticidad de la vulnerabilidad. Por ejemplo, en los laboratorios de XSS, la mayoría de los retos te pedirán simplemente generar una alerta con `alert()`. Además de completar ese paso, trata de pensar cómo podrías robar la cookie de un usuario, qué técnicas de codificación podrías aplicar, o de qué otras maneras podrías lograr el mismo resultado. Este enfoque más profundo te ayudará a desarrollar una mentalidad ofensiva más sólida y aplicable a escenarios reales.
- Una vez que hayas completado todos los laboratorios y tengas tu *cheat sheet* lista, es momento de enfrentarte a los *Mystery Labs* que ofrece PortSwigger. En estos desafíos, se te presenta un laboratorio sin ningún contexto ni indicación sobre la vulnerabilidad que debes encontrar, lo cual los hace ideales para poner a prueba tu metodología y la efectividad de tu *cheat sheet*. PortSwigger recomienda realizar al menos 5 *Mystery Labs* antes de presentar el examen. En mi caso, llegué a completar fácilmente más de 200, pero eso depende del ritmo y nivel de confianza de cada persona. Lo importante es que los practiques hasta sentirte realmente cómodo resolviéndolos por tu cuenta**.**

![image.png](/assets/img/BSCPExam/image%206.png)

- Algo que no me gustó de los *Mystery Labs* es que, después de cierto punto, tienden a repetirse bastante y no resultan tan aleatorios como uno desearía. Para solucionar esto, utilicé un script en Python que extrae todos los laboratorios de PortSwigger en un archivo HTML —puedes encontrarlo en el siguiente [enlace](https://github.com/ossamayasserr/PortSwigger-Labs-in-1-Page)— y, con la ayuda de ChatGPT, desarrollé un pequeño script adicional que selecciona un laboratorio al azar para practicar de forma más dinámica.
- Llegado a este punto, probablemente ya estés casi listo para rendir el examen BSCP. La mejor forma de poner a prueba tus conocimientos, tu metodología y tu *cheat sheet* es resolviendo los *Practice Exams* que proporciona PortSwigger. Estos exámenes de práctica están divididos en tres fases, al igual que el examen real, lo que los convierte en una excelente herramienta para evaluar tu preparación de forma realista.
- A continuación, comparto mi *write-up* del *Practice Exam*, pero te recomiendo leerlo solo si ya lo resolviste por tu cuenta, para evitar posibles *spoilers*. Realizar el *Practice Exam* sin ayuda y al 100% por tu cuenta es, sin duda, la mejor manera de medir tu nivel actual y saber si estás verdaderamente preparado para enfrentar el examen oficial.

## Extensiones de Burp

- A medida que avanzas y resuelves los laboratorios en la Web Security Academy, se te van sugiriendo distintas extensiones para Burp Suite. A continuación, te comparto las que utilicé durante mi preparación y en le examen:
    - [Param Miner](https://portswigger.net/bappstore/17d2949a985c4b7ca092728dba871943)
    - [**HTTP Request Smuggler**](https://portswigger.net/bappstore/aaaa60ef945341e8a450217a54a11646)
    - [**Content Type Converter**](https://portswigger.net/bappstore/db57ecbe2cb7446292a94aa6181c9278)
    - [**JWT Editor**](https://portswigger.net/bappstore/26aaa5ded2f74beea19e2ed8345a93dd)
    - [**Server-Side Prototype Pollution Scanner**](https://portswigger.net/blog/server-side-prototype-pollution-scanner)
    - [**DOM Invader**](https://portswigger.net/burp/documentation/desktop/tools/dom-invader)
    - [**Java Deserialization Scanner**](https://portswigger.net/bappstore/228336544ebe4e68824b5146dbbd93ae)**
    - [**Hackvertor**](https://portswigger.net/bappstore/65033cbd2c344fbabe57ac060b5dd100)

## Mi experiencia con el Examen

Para ser honesto, aprobé la certificación en mi segundo intento. El primero fue un tanto desastroso: no logré superar ninguna de las tres fases en ninguna de las dos aplicaciones. Creo que esto se debió a varios factores. Aunque había completado muchos *Mystery Labs* y me sentía confiado identificando vulnerabilidades, me di cuenta de que no tenía los conceptos tan interiorizados como creía. En muchos casos resolvía los laboratorios de forma mecánica, sin razonar realmente por qué ciertas técnicas funcionaban o dejaban de funcionar. No analizaba a fondo por qué una codificación específica hacía que un exploit fuera exitoso, y eso me pasó factura.

Ese primer intento fue clave para detectar mis vacíos de conocimiento y comenzar a trabajar en ellos de manera más consciente. Me di cuenta de que, si bien podía seguir los pasos para resolver un lab, me faltaba una comprensión más profunda del 'por qué' detrás de ciertas explotaciones, especialmente en áreas como:

* **Codificaciones y Ofuscación:** A menudo, no entendía completamente por qué una técnica de codificación específica hacía que un exploit fuera exitoso o por qué fallaba con otra. Para esto, además de la guía de PortSwigger sobre ofuscación que ya mencioné ([https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings](https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings)), me enfoqué en practicar escenarios donde la sanitización de la entrada era estricta, forzándome a probar diversas combinaciones de codificación (URL, HTML, Unicode, etc.) hasta que el payload pasara.
* **Lógica de Negocio y Vulnerabilidades 'Chain':** Noté que, aunque podía encontrar vulnerabilidades individuales, me costaba más encadenarlas para lograr un objetivo mayor (como escalar privilegios). Para esto, los Mystery Labs de la Web Security Academy fueron invaluables, pero también busqué write-ups de Bug Bounties complejos en HackerOne y otras plataformas, prestando especial atención a cómo los investigadores combinaban hallazgos menores para lograr un impacto crítico.
* **Uso Avanzado de Extensiones de Burp:** Como mencioné, no tener mis extensiones de Burp Suite correctamente configuradas o entender su alcance real me perjudicó. Dediqué tiempo a leer la documentación de cada extensión clave (Param Miner, JWT Editor, etc.) y a probar sus funcionalidades en los labs para entender cómo optimizar su uso y no depender solo del escaneo automático.

Otro factor que influyó negativamente fue la falta de preparación y configuración adecuada de mis extensiones en Burp Suite. Durante mis prácticas utilizaba un entorno virtual con Kali Linux, pero para el examen instalé la versión Pro de Burp Suite en Windows. Para mi sorpresa, varias extensiones no funcionaban igual de bien o tardaban mucho más en detectar vulnerabilidades en comparación con Linux. Esto fue un error crítico, ya que al tratarse de un examen de cuatro horas, cada minuto cuenta, y el examen está diseñado para que aproveches Burp y sus extensiones de la forma más eficiente posible, muchas veces automatizando parte del trabajo.

Por suerte, el entorno de las aplicaciones funcionó de forma estable y rápida, sin ningún tipo de conflicto técnico.

### Intento 2

Para mi segundo intento ya había identificado las vulnerabilidades en las que tenía mayores vacíos y me aseguré de corregirlos. Gracias a ello, me sentía mucho más preparado y confiado. Esta vez tenía todas mis herramientas listas y correctamente configuradas, y opté por usar un entorno virtualizado con Kali Linux 2025.

Mi estrategia fue auditar manualmente una de las aplicaciones mientras en la otra dejaba ejecutándose los escaneos automáticos de Burp Suite. Identifiqué la primera vulnerabilidad rápidamente, aunque su explotación me tomó algo de tiempo. A los 40 minutos de haber comenzado el examen ya había completado la Fase 1 en una de las aplicaciones. Sin embargo, fue en ese momento cuando comenzaron los problemas: el entorno del laboratorio empezó a comportarse de manera inestable, las aplicaciones se volvieron extremadamente lentas y apenas podía interactuar con ellas.

Intenté cambiar de navegador, reiniciar Burp Suite, e incluso consideré que podía ser un problema de mi conexión, pero al comprobar que esta era estable, concluí que el problema venía directamente del entorno del examen. En medio de la frustración, decidí redactar un correo al equipo de PortSwigger informando la situación, pero debido a la diferencia horaria, no se encontraban en horario laboral.

Este escenario fue muy desmotivador, ya que en mi primer intento el entorno había funcionado sin inconvenientes. Además, coincidió con otros problemas técnicos que había notado en los laboratorios de la plataforma durante esos días, lo que me hizo sospechar que se trataba de una inestabilidad general del sistema. Para entonces ya había perdido cerca de una hora, lo que me dejaba con solo dos horas para completar el resto del examen.

Afortunadamente, al inicio de la tercera hora las aplicaciones comenzaron a responder correctamente, y pude continuar con la auditoría. Aunque era consciente de que probablemente no tendría tiempo suficiente para completar todo el examen, decidí mantenerme enfocado y no dejar que la situación me desmotivara. La escalada de privilegios y la lectura del archivo interno me tomó cerca de una hora. Eso significaba que apenas me quedaba poco más de una hora para abordar la segunda aplicación.

Me mantuve con la mejor actitud posible, y por suerte, en la segunda aplicación identifiqué rápidamente la ruta de explotación. Gracias a eso, logré completarla con éxito. Al final, terminé ambas aplicaciones con apenas diez minutos restantes antes de que se agotaran las cuatro horas del examen.

## Tips

- Una de las claves fundamentales para completar exitosamente el examen es contar con una *cheat sheet* bien estructurada y completa, que incluya todas las vulnerabilidades abordadas en la Academy junto con sus diferentes técnicas de explotación. Dado que el examen tiene una duración de solo cuatro horas, el tiempo puede resultar muy limitado, y tener esta referencia a mano te permitirá ahorrar valiosos minutos y mantener un enfoque más eficiente.

- La gestión del tiempo es, sin duda, uno de los factores más críticos para el éxito en el examen BSCP. Con solo 4 horas disponibles para dos aplicaciones y tres fases por cada una, cada minuto cuenta. Basado en mi experiencia, y especialmente en mi segundo intento donde el tiempo fue un desafío, recomiendo la siguiente distribución aproximada de tiempo por aplicación y fase:
    * **Fase 1 (Acceso a cuenta estándar):** Intenta completar esta fase en la primera aplicación en unos **45-60 minutos**. Una vez lograda, replica la metodología o busca la vulnerabilidad inicial en la segunda aplicación. Es vital no quedarse atascado aquí, ya que sin este acceso, las fases posteriores son imposibles. 
    * **Fase 2 (Escalada de privilegios a administrador):** Esta fase puede tomar un poco más de tiempo. Apunta a unos **60-75 minutos** por aplicación. Aquí es donde entra en juego la creatividad y el conocimiento más profundo de las vulnerabilidades de control de acceso, lógica de negocio, etc.
    * **Fase 3 (Lectura de archivo local):** Esta suele ser la fase más desafiante y donde el pensamiento "fuera de la caja" es clave. Prevé unos **45-60 minutos** por aplicación. Las *web shells*, inyecciones de comandos, o deserializaciones pueden ser el camino aquí.
    * Un consejo crucial es **no te quedes atascado**. Si llevas más de 20-30 minutos en una vulnerabilidad sin progreso claro, anótala y pasa a otra posible vía. A veces, cambiar de enfoque o incluso pasar a la otra aplicación puede refrescar tu perspectiva. Burp Suite te permite alternar fácilmente entre proyectos, así que aprovecha la posibilidad de dejar escaneos activos en una aplicación mientras trabajas manualmente en otra.

- Si estás seguro de que estás realizando la explotación de forma correcta pero no logras que funcione, prueba con diferentes técnicas de ofuscación. Muchas veces, pequeños cambios en la codificación o en la forma en que se envía la carga pueden marcar la diferencia. Por eso, te recomiendo revisar la siguiente guía, donde se recopilan diversas técnicas de ofuscación útiles para este tipo de escenarios:
    - [https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings](https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings)
- En caso de no aprobar la certificación, asegúrate de guardar el archivo del proyecto de Burp Suite. Esto te permitirá revisarlo con calma más adelante y analizar con detalle las peticiones, respuestas y flujos de la aplicación. De esa manera, podrás identificar vectores de ataque que tal vez pasaste por alto durante el examen y aprender de tus errores para mejorar en el siguiente intento.
- Una técnica muy útil para comprobar si realmente dominas un tema es intentar enseñárselo a otra persona, o incluso explicarlo en voz alta como si lo estuvieras haciendo. Al verbalizar los conceptos, te darás cuenta rápidamente de los vacíos que puedas tener y podrás reforzar esas áreas con mayor eficacia.
- Trata siempre de pensar "fuera de la caja"; desarrollar esta habilidad es fundamental para convertirnos en buenos analistas de seguridad. Muchas veces, las vulnerabilidades más interesantes o críticas no están a simple vista, y es ese pensamiento creativo y lateral el que marca la diferencia en escenarios reales.
- Te recomiendo encarecidamente explorar la sección de investigación de PortSwigger ([https://portswigger.net/research](https://portswigger.net/research)) y ver los videos que allí se comparten. En particular, la charla de James Kettle sobre *HTTP Request Smuggling*, presentada en DEFCON, es simplemente magnífica. No solo es sumamente didáctica, sino que también resulta muy inspiradora, ya que demuestra cómo muchos de los laboratorios disponibles en la Academy están basados en escenarios reales descubiertos durante investigaciones del equipo de PortSwigger.
- Creo que siempre es fundamental mantener una *mentalidad de crecimiento* —no solo para esta certificación, sino para cualquier desafío en general—, especialmente en un campo como la ciberseguridad, donde todo evoluciona rápidamente. Mantente abierto al aprendizaje constante, acepta los errores como parte del proceso, y, sobre todo, trata de disfrutar cada etapa del camino. Al final, lo más valioso no es solo la certificación, sino todo lo que aprendes durante el recorrido.

## Conclusión

La certificación BSCP fue un desafío exigente, pero sumamente enriquecedor. Disfruté cada etapa del proceso, desde la preparación hasta la obtención del certificado. A lo largo del camino adquirí un conocimiento profundo sobre hacking web y AppSec, por lo que sin duda la recomendaría a cualquier persona que desee desarrollarse en estas áreas.

Como en todo reto, enfrenté obstáculos y momentos de frustración, pero estoy convencido de que lo más importante es mantener una actitud positiva, tener una mentalidad de crecimiento y disfrutar del proceso de aprendizaje.

### Aplicación Práctica y Futuro

Ahora que he obtenido la certificación BSCP, mi objetivo principal es aplicar todo este conocimiento de forma práctica. No se trata solo de tener un certificado, sino de la capacidad de identificar y explotar vulnerabilidades en escenarios del mundo real.

* **En Pentesting:** La metodología y la profundidad en el uso de Burp Suite aprendidas durante la preparación del BSCP son directamente transferibles a mis proyectos de pruebas de penetración web. Me siento mucho más seguro al abordar aplicaciones complejas, identificando vectores de ataque menos obvios y elaborando exploits más sofisticados.
* **En Programas de Bug Bounty:** Esta certificación ha fortalecido mi 'ojo' para encontrar vulnerabilidades y mi capacidad para reportarlas de manera efectiva, incrementando mis posibilidades de éxito en plataformas como HackerOne o Bugcrowd. La mentalidad de 'ir un paso más allá' que desarrollé al resolver los labs es fundamental para descubrir hallazgos únicos.
* **Desarrollo Profesional Continuo:** El campo de la ciberseguridad web evoluciona constantemente. El BSCP me ha proporcionado una base sólida para seguir aprendiendo sobre nuevas técnicas de ataque y defensa, manteniéndome relevante en esta área. Mi próximo paso es seguir explorando vulnerabilidades más avanzadas y contribuir a la comunidad compartiendo mis hallazgos y conocimientos.

Espero que hayas disfrutado esta review y que te haya sido útil en tu preparación para la certificación. Si deseas contactarme o tienes alguna duda, tienes a disposición mis redes sociales.

¡Happy Hacking! 🔐💻

## Recursos

### Cheet Sheet:

- [https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study)
- [https://github.com/DingyShark/BurpSuiteCertifiedPractitioner](https://github.com/DingyShark/BurpSuiteCertifiedPractitioner)
- [https://bscp.guide/](https://bscp.guide/)

### Youtube / Courses:

- S4vitar & Hack4u: en mi opinión, ofrecen el mejor contenido en español. S4vitar siempre trata de ir un paso más allá al enfrentarse a una máquina o laboratorio, lo que se refleja en la profundidad de su enfoque. Su nuevo curso *Hacking Web*, disponible en su academia, en el que resuelve todos los laboratorios de PortSwigger, es una excelente herramienta para prepararte específicamente para esta certificación.
    - [https://www.youtube.com/@S4viSinFiltro](https://www.youtube.com/@S4viSinFiltro)
    - [https://hack4u.io/](https://hack4u.io/)
- Gonzalo Carrasco: excelente para aprender sobre OAuth vulnerabilidades
    - [https://www.youtube.com/@gonzalocarrasco](https://www.youtube.com/@gonzalocarrasco)
- Rana Khalil: Explica de forma excelente y muy didáctica, lo cual es perfecto para mejorar tu comprensión y aprender cómo aplicar una metodología efectiva frente a las distintas clases de vulnerabilidades.
    - [https://www.youtube.com/@RanaKhalil101](https://www.youtube.com/@RanaKhalil101)
    - [https://academy.ranakhalil.com/](https://academy.ranakhalil.com/)
- Jarno Timmermans: excelente para aprender sobre HTTP Request Smuggling
    - [https://www.youtube.com/@netletic](https://www.youtube.com/@netletic)
- Seven Seas Security: excelente para aprender sobre SSTI and XXE
    - [https://www.youtube.com/@7SeasSecurity/playlists](https://www.youtube.com/@7SeasSecurity/playlists)

### Otros recursos:

- [https://bscpcheatsheet.gitbook.io/exam](https://bscpcheatsheet.gitbook.io/exam)
- [https://deephacking.tech/burp-suite-certified-practicioner-review-bscp/](https://deephacking.tech/burp-suite-certified-practicioner-review-bscp/)
- [https://github.com/ricardojoserf/Portswigger-Labs](https://github.com/ricardojoserf/Portswigger-Labs)
- [https://h0j3n.github.io/posts/Burpsuite-Practice-Exam/#walkthrough-33](https://h0j3n.github.io/posts/Burpsuite-Practice-Exam/#walkthrough-33)
- [https://medium.com/@ossamayasserr/how-to-crush-bscp-exam-in-75-mins-bscp-review-0b207a17e26d](https://medium.com/@ossamayasserr/how-to-crush-bscp-exam-in-75-mins-bscp-review-0b207a17e26d)
- [https://github.com/ossamayasserr/BSCP-Notes](https://github.com/ossamayasserr/BSCP-Notes)
- [https://www.youtube.com/watch?v=w-eJM2Pc0KI&ab_channel=DEFCONConference](https://www.youtube.com/watch?v=w-eJM2Pc0KI&ab_channel=DEFCONConference)
- [https://portswigger.net/research/http2](https://portswigger.net/research/http2)