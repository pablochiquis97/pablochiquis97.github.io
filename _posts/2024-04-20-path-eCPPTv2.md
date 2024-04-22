---
title: Path de estudio para el eCPPTv2
date: 2024-04-22 14:10:00 +0800
author: pablo
categories: [certifications, eCPPTv2]
tags: [
    eLearnSecurity
    Review
    Pentesting
    eCPPTertification,
  ]
image:
  path: /assets/img/eCPPT/ecPPTv2Certication.png
---

## Descripción

Recientemente, logré aprobar con éxito la certificación eCPTTv2. La preparación la realicé de manera autodidacta, aprovechando las máquinas de Hack The Box (HTB), contenido creado por la comunidad y diversos recursos disponibles en línea por lo que en este post, compartiré mi experiencia sobre cómo me preparé, qué recursos recomiendo y todo lo que necesitas saber si estás considerando presentarte al examen. Además, si estás interesado en conocer más detalles sobre mi experiencia con la certificación, encontrarás una [Review](https://pablochiquis97.github.io/posts/eCPPTv2/) detallada que preparé.

---

## Proceso de Estudio y Preparación

Como mencione en la [Review](https://pablochiquis97.github.io/posts/eCPPTv2/) los conocimientos más importantes que considero que debes tener en cuenta si te vas a presentar al examen son los siguientes:

- Hacking web
- Enumeración
- Técnicas de Post-explotación en sistemas Windows y Linux
- Escalada de Privilegios Linux
- Buffer Overflow
- Pivoting

La ruta que yo elegí para la preracion del examen es la que recomienda "ViejoFraile" en su video:

- [Road to eCPPTv2](https://www.youtube.com/watch?v=2wRFI65Cn7A&ab_channel=ViejoFraile)

En el video, se sugieren los filtros que se deben utilizar para buscar las máquinas correspondientes en el buscador de [máquinas](https://infosecmachines.io/) de S4vitar.

Filtros para encontrar las máquinas mencionadas:

- ejpt hackthebox linux
- hackthebox windows easy
- hackthebox eCPPT easy
- hackthebox eCPPT medium

## Hacking Web

Para el tema de Hacking Web te recomiendo realizar las siguientes máquinas:

- [Goodgames](https://pablochiquis97.github.io/posts/goodgames/)
- [Validation](https://pablochiquis97.github.io/posts/validation/)
- [Schooled](https://pablochiquis97.github.io/posts/maquina-schooled/)
- [My Expense](https://pablochiquis97.github.io/posts/myExpense/) de Vulnhub

Te sugiero estar al tanto de cómo funciona SQLMap, ya que proporciona un formato limpio y ordenado, especialmente útil para mostrar evidencia en caso de realizar un informe.

## Técnicas de Post-explotación

Para las técnicas de Post-explotación te recomiendo las siguientes máquinas:

- [Blue](https://pablochiquis97.github.io/posts/maquina-blue/)
- [Resolute](https://pablochiquis97.github.io/posts/maquina-resolute/)

Ambas son máquinas Windows. En una de ellas, se explota la vulnerabilidad del EternalBlue, tanto de manera manual como automática a través de Metasploit. Aunque esta vulnerabilidad es relativamente directa de explotar, es importante dominar su explotación. Una vez que se tiene control sobre el sistema, se cubren diversas técnicas de post-explotación. Entre ellas, recomiendo especialmente revisar cómo habilitar el servicio RDP y obtener una interfaz gráfica, ya que esto será muy útil durante el examen.

## Escalada de Privilegios

En todas las máquinas de Hack The Box, una vez se logra acceso, sigue la fase de obtener privilegios, como el usuario root. Por lo tanto, con cada máquina, estarás practicando la escalada de privilegios. Te recomiendo que vayas más allá de simplemente aprender cómo utilizar los diferentes exploits. Es importante comprender la metodología y el proceso de enumeración para obtener un vector de ataque que te permita convertirte en el usuario root.

También es recomendable que te familiarices con herramientas como:

- [PSPY](https://github.com/DominicBreuker/pspy)
- [linPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS)
- [winPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS)

## Buffer Overflow

Para el tema de buffer overflow, te recomiendo revisar el writeup de la máquina [Buff](https://pablochiquis97.github.io/posts/maquina-buff/), donde expliqué detalladamente la metodología a seguir para llevar a cabo el buffer overflow. Además, otras maquinas para practicar son:

- Máquina Gatekeeper THM - [https://tryhackme.com/room/gatekeeper](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbko4aXpFVkw5NU5BVDZNYXI1U0lvWERMdWRuZ3xBQ3Jtc0tuV2NDSDVVZHpXNUs3Z2d1VVA1M2lfd29IOGVISnlzTVdVX0ZrcFVfRkExWjhpVTFLU1lrR3NydXIyMW9QMHd6d3VtN2lNNkJGM0FwWDdYYjRRSVNDXzRNSlE2THNZSEV6YnQ1X01HRF8zbWk5eXFXbw&q=https%3A%2F%2Ftryhackme.com%2Froom%2Fgatekeeper&v=2wRFI65Cn7A)
- Máquina Brainpan THM - [https://tryhackme.com/room/brainpan](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqazZLUjZmSUlRUEJqdFY4YUNqT2lKNWs2Wjk5Z3xBQ3Jtc0ttSlVHbm82SW41NldkZkh0OG5wbkpVTTZOeER1U2NVTk8wS0NxZmNPVnZldFhhNHl3cGpDR2duVVZyQldWMWFyNDNWSkc1SWw4Zi1Ec3RYLWZHb09wVERFTzFyNHlZLWREeGxQeU9NSml3UzZyVkJrQQ&q=https%3A%2F%2Ftryhackme.com%2Froom%2Fbrainpan&v=2wRFI65Cn7A)

Los siguientes videos son recomendables para comprender la metodología a seguir. Una vez dominada, el proceso se vuelve muy monótono.

- [¿Cómo explotar el Buffer Overflow del OSCP con éxito?](https://www.youtube.com/watch?v=sdZ8aE7yxMk&ab_channel=s4vitar)
- [Buffer Overflows Made Easy (2022 Edition)](https://www.youtube.com/watch?v=ncBblM920jw&t=463s&ab_channel=TheCyberMentor)

## Pivoting

El pivoting es una parte central del examen, por lo que es crucial dominar este tema. En mi caso, realicé el pivoting manual a través de Chisel y Socat. Sin embargo, te recomiendo también tener contemplada otra opción en caso de que falle el pivoting manual, como realizar el pivoting a través de Metasploit. Recursos recomendados:

- [Redish](https://pablochiquis97.github.io/posts/maquina-reddish/) es una máquina clasificada como dificultad Insane, pero mas que centrarte en como realizar la explotación te recomiendo que hagas énfasis en practicar el pivoting, entender como descubrir nuevos hosts, transferencia de archivos entre redes y tener clara la distribución del esquema de red.

S4vitar dispone de varios videos que explican detalladamente el tema del pivoting y cómo montar tus propios laboratorios. Te recomiendo realizar todos estos ejercicios. Una vez que revises todo este material, el tema del pivoting estará más que dominado.

- [Playlist Pivoting S4vitar](https://www.youtube.com/playlist?list=PLT5iG9p-ZFahO5_9jtw6OEfkjoXMF2elz)

## Redacción del Informe

Para la redacción del informe, te recomiendo que vayas tomando capturas de pantalla y evidencia mientras realizas el examen. Mantener un orden y una estructura organizada es fundamental.

El equipo de INE no impone restricciones en cuanto a la longitud del informe, solo proporciona ciertas pautas que debes seguir. En mi caso, mi informe ocupó 65 páginas, pero revisé en varios reviews que a otros les salió mucho menos o mucho más. Por lo tanto, la forma en que redactes tu informe es muy subjetiva. Te recomiendo hacer énfasis en la criticidad de las vulnerabilidades, la evidencia de concepto (PoC), el impacto y el riesgo, el puntaje CVSS y las recomendaciones de remediación. Recursos recomendados:

- [Writing a Pentest Report](https://www.youtube.com/results?search_query=report+pentest+tcm)
- [APRENDE a HACER un Reporte Profesional](https://www.youtube.com/watch?v=hmQhd8fUyKk&t=914s&ab_channel=Jackie0x17)

## Simulaciones del Examen

Una vez que hayas revisado todos los temas, te sugiero realizar las simulaciones del examen.

- [Simulación eCPPTV2 - Securiters](https://www.youtube.com/playlist?list=PLXiewoasm8Vn0ByHo8pDKC17Pn4IfZMyt)

* [Simulación de examen eCPPTv2 - S4vitar](https://www.youtube.com/watch?v=Q7UeWILja-g&t=20028s&ab_channel=S4viOnLive%28BackupDirectosdeTwitch%29)

Las chicas del canal de Securiters, en colaboración con "elHackerEtico", configuraron su propio entorno del eCPPT. Considero que el nivel de dificultad de las máquinas codificadas en este laboratorio está muy próximo a lo que se toca en el examen real.

Por otro lado, S4vitar nos ofrece un laboratorio con seis máquinas, donde explica de manera detallada el concepto de Pivoting entre redes, la transferencia de archivos tanto entre máquinas Windows como Linux, y otros consejos importantes a tener en cuenta durante el examen. En cuanto al nivel de dificultad de las máquinas, considero que son mucho más desafiantes que lo que encontrarás en el entorno real del examen. Sin embargo, si logras comprender completamente todas las soluciones del laboratorio, no deberías tener problemas para obtener la certificación.

## Otros recursos interesantes:

- [Pivoting con Chisel, Socat y Metasploit](https://www.youtube.com/watch?v=9m9R_7KU5q4&t=2069s&ab_channel=Cryproot)
- [Doble Pivoting](https://www.youtube.com/watch?v=zGm7kUvC31M&t=1518s&ab_channel=Jackie0x17)
- [Explore Hidden Networks With Double Pivoting](https://pentest.blog/explore-hidden-networks-with-double-pivoting/)
- [Que es la politica LocalAccountTokenFilterPolicy?](https://autonomiahacker.com/index.php/2023/05/24/localaccounttokenfilterpolicy/)
- [Tunneling and Pivoting](https://0xdf.gitlab.io/2019/01/28/pwk-notes-tunneling-update1.html)

Sin más que agregar, espero que hayas disfrutado del post y que los consejos y recomendaciones te sean útiles en caso de presentarte a la certificación.
