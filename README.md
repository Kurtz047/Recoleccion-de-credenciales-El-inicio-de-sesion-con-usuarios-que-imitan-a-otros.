# Recoleccion de credenciales: El inicio de sesion con usuarios que imitan a otros
Investigar una campaña de phishing de credenciales con adversario en el medio (AiTM) que atrajo a un empleado a una página de inicio de sesión de Microsoft 365 que imitaba a la original. Trabajando exclusivamente con registros SIEM, identificará el dominio falso, reconstruirá la cadena de redireccionamiento de múltiples saltos a través de un sitio legítimo comprometido, descubrirá una segunda oleada de phishing contra otro empleado y confirmará la toma de control de la cuenta en los registros de inicio de sesión de Azure AD mediante la repetición de tokens de sesión y el análisis de rutas imposibles. Finalizará seleccionando la acción de contención que expulsa al atacante que posee un token de sesión válido. Habilidades fundamentales para analistas de SOC en análisis de dominios falsos y respuesta a incidentes centrada en la identidad.

Bienvenidos al SOC. Hemos recibido una alerta de nuestro sistema de correo electrónico: la empleada Alicia Rodríguez, en la estación de trabajo corp-wks-7712, hizo clic en un enlace de un correo electrónico sospechoso que la redirigió a lo que parecía ser una página de inicio de sesión de Microsoft 365. Sus credenciales podrían haber sido comprometidas.

# ¿Qué es la recolección de credenciales AiTM?
El phishing de intermediario (AiTM, por sus siglas en inglés) es una técnica en la que los atacantes interponen un servidor proxy entre la víctima y el servicio de autenticación legítimo. La víctima ve una página de inicio de sesión convincente, a menudo en un dominio similar, e introduce sus credenciales. El proxy las redirige al servicio auténtico y guarda una copia de toda la información que recibe, incluyendo los datos de la sesión que el servicio devuelve una vez superada la autenticación multifactor.

¿Por qué son tan efectivos los dominios similares? Un nombre de dominio se lee de derecha a izquierda, pero el ojo lee de izquierda a derecha, y los atacantes se centran en la percepción visual. Toman prestada una palabra clave de una marca reconocida, la colocan donde se dirige la mirada y la asocian a un nombre que ellos mismos registraron. Las sustituciones de caracteres, los subdominios adicionales y las terminaciones desconocidas se confunden rápidamente. El hábito que los neutraliza es mecánico: leer primero la terminación, porque es ahí donde se indica quién opera realmente el nombre.

# Lo que sabemos hasta ahora
Usuario afectado: alicia.rodriguez (estación de trabajo corp-wks-7712)
Se le enviaron dos correos electrónicos de phishing desde remitentes externos a su bandeja de entrada.
Al menos un correo electrónico contenía un enlace a una página de inicio de sesión que imitaba a la original.
Nuestros sistemas de inteligencia sobre amenazas han detectado múltiples dominios e IP sospechosos.
Los registros de inicio de sesión de Azure AD muestran actividad anómala en la cuenta de Alicia.
En este tutorial, seguiremos la cadena de ataque completa: desde los correos electrónicos de phishing, pasando por la infraestructura maliciosa, hasta el robo de credenciales y los intentos de movimiento lateral. Nuestras herramientas son los registros SIEM de siete fuentes diferentes en todo nuestro entorno.

# El dominio de phishing
Esa mañana, Alicia recibió dos mensajes en su bandeja de entrada, entre el tráfico habitual de un día laborable. Uno de ellos contenía un enlace con sus credenciales alojadas en el servidor de otra persona. En algún lugar de los registros que aparecen a continuación se encuentra el dominio que suplantaba la identidad de una página de inicio de sesión de Microsoft 365.

Registros de puerta de enlace de correo electrónico y DNS
El fragmento combina las entregas de correo electrónico a través de la puerta de enlace con las consultas DNS que la estación de trabajo de Alicia realizó en la misma ventana. También se incluye el tráfico corporativo habitual, y ese es el punto clave: determinar qué líneas pertenecen al incidente es el trabajo.
