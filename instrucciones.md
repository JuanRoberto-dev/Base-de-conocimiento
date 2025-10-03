# Sistema de Inscripción de Cursos

Este proyecto permite que un alumno pueda seleccionar cursos respetando la seriación y los prerrequisitos de cursos anteriores, usando Tau Prolog en el navegador.

## Tau Prolog

Tau Prolog es un intérprete de Prolog del lado del cliente implementado en JavaScript, compatible con navegadores. Permite ejecutar lógica Prolog dentro de un HTML, consultando bases de conocimiento y mostrando resultados dinámicamente en la página.

### Descargar Tau Prolog

Se puede descargar la librería desde GitHub o npm:

- [GitHub Tau Prolog](https://github.com/tau-prolog/tau-prolog)  
- [npm Tau Prolog](https://www.npmjs.com/package/tau-prolog)

En este proyecto, basta con colocar `tau-prolog.js` en la misma carpeta que el HTML.

## Archivos del proyecto

- `cursos.html` → Contiene la interfaz, los scripts JavaScript y la base de conocimiento de los cursos.  
- `estilos.css` → Contiene el diseño visual de la página.  
- `tau-prolog.js` → Librería externa para ejecutar Prolog en el navegador.

## Instalación / Preparación

1. Descargar los archivos del proyecto.  
2. Asegurarse de que `cursos.html`, `estilos.css` y `tau-prolog.js` estén en la misma carpeta.  
3. No se requiere instalación adicional ni compilación.

## Ejecutar el proyecto

1. Abrir `cursos.html` con un navegador moderno (Chrome, Firefox, Edge).  
2. Seleccionar un alumno del menú desplegable.  
3. Presionar **“Ver cursos disponibles”**.  
4. Los cursos se mostrarán en tres secciones:

   - Cursos disponibles (verde).  
   - Cursos bloqueados por prerrequisitos (rojo).  
   - Cursos bloqueados por repetición (naranja).

5. Seleccionar los cursos deseados y hacer clic en **“Confirmar selección”** para ver la selección final.

## Código completo del proyecto

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Selección de Cursos</title>
    <script src="tau-prolog.js"></script>
    <link rel="stylesheet" href="estilos.css">
</head>
<body>
    <h2>Sistema de Inscripción de Cursos</h2>
    <p>Seleccione un alumno:</p>
    <select id="alumno">
        <option value="ana">Ana</option>
        <option value="juan">Juan</option>
        <option value="luis">Luis</option>
    </select>
    <br><br>
    <input type="button" value="Ver cursos disponibles" onclick="consultaCursos()"/>
    <h3>Cursos disponibles:</h3>
    <div id="disponibles"></div>
    <h3>Cursos bloqueados por prerrequisitos:</h3>
    <div id="bloqueados"></div>
    <h3>Cursos bloqueados por repetición:</h3>
    <div id="repetidos"></div>

    <!-- Base de conocimiento Prolog -->
    <script id="cursos.pl" type="text/prolog">
        curso(matematicas1).
        curso(matematicas2).
        curso(programacion1).
        curso(programacion2).
        curso(logica).
        curso(inteligencia_artificial).

        prerequisito(matematicas2, matematicas1).
        prerequisito(programacion2, programacion1).
        prerequisito(inteligencia_artificial, logica).
        prerequisito(inteligencia_artificial, programacion2).

        aprobado(ana, matematicas1).
        aprobado(ana, programacion1).
        aprobado(juan, matematicas1).
        aprobado(juan, programacion1).
        aprobado(juan, programacion2).
        aprobado(luis, matematicas1).

        puede_cursar(Alumno, Curso) :-
            curso(Curso),
            forall(prerequisito(Curso, Pre), aprobado(Alumno, Pre)).
    </script>

    <script>
        function consultaCursos(){
            var alumno = document.getElementById("alumno").value;
            var disponibles = document.getElementById("disponibles");
            var bloqueados = document.getElementById("bloqueados");
            var repetidos = document.getElementById("repetidos");
            disponibles.innerHTML = "";
            bloqueados.innerHTML = "";
            repetidos.innerHTML = "";

            var session = pl.create(1000);
            var programa = document.getElementById("cursos.pl").text;
            session.consult(programa);

            // Buscar todos los cursos
            session.query("curso(C).");
            session.answers(function(ans){
                if(pl.type.is_substitution(ans)){
                    let curso = ans.lookup("C");

                    // Subsession para comprobar prerrequisitos y repetición
                    var subSession = pl.create(1000);
                    subSession.consult(programa);

                    // Ver si ya aprobado
                    subSession.query("aprobado(" + alumno + "," + curso + ").");
                    subSession.answer(function(r){
                        if(pl.type.is_substitution(r)){
                            repetidos.innerHTML += "<p class='repetido'>" + curso + " (ya aprobado)</p>";
                        } else {
                            // Ver si puede cursar
                            var sub2 = pl.create(1000);
                            sub2.consult(programa);
                            sub2.query("puede_cursar(" + alumno + "," + curso + ").");
                            sub2.answer(function(res){
                                if(pl.type.is_substitution(res)){
                                    disponibles.innerHTML += "<p class='ok'>" + curso + "</p>";
                                } else {
                                    bloqueados.innerHTML += "<p class='no'>" + curso + " (falta prerrequisito)</p>";
                                }
                            });
                        }
                    });
                }
            });
        }
    </script>
</body>
</html>