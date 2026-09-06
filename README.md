📊 Análisis de RR.HH.: Rotación y Desempeño de Empleados
![SQL Server](https://img.shields.io/badge/SQL%20Server-2019-CC2927?logo=microsoftsqlserver&logoColor=white)
![Status](https://img.shields.io/badge/Estado-Completado-brightgreen)
📑 Tabla de Contenidos
Resumen del Proyecto
Descripción de las Columnas
Análisis
Conclusiones
---
📋 Resumen del Proyecto
El personal de RR.HH. de la empresa GreatPlaceToWork (caso de estudio ficticio planteado para este proyecto) desea mejorar el desempeño, aumentar la retención y mejorar la satisfacción laboral general de sus empleados. Sin embargo, no cuenta con una visión clara de los datos pertinentes.
El objetivo de este proyecto es utilizar SQL dentro de SQL Server Management Studio para analizar los datos disponibles y proporcionar recomendaciones al departamento de RR.HH. que faciliten mejoras exitosas.
Fuente de datos: Employee Attrition Prediction Dataset — Kaggle (10,000 registros sintéticos de empleados, usados con fines educativos/de práctica)
---
🗂️ Descripción de las Columnas
Columna	Descripción
`Employee_ID`	Identificador único del empleado
`Age`	Edad del empleado
`Gender`	Género
`Marital_Status`	Estado civil
`Department`	Departamento al que pertenece
`Job_Role`	Puesto/rol de trabajo
`Job_Level`	Nivel jerárquico del puesto
`Monthly_Income`	Ingreso mensual
`Hourly_Rate`	Tarifa por hora
`Years_at_Company`	Años de antigüedad en la empresa
`Years_in_Current_Role`	Años en el puesto actual
`Years_Since_Last_Promotion`	Años desde su último ascenso
`Work_Life_Balance`	Balance vida-trabajo (escala 1-4)
`Job_Satisfaction`	Satisfacción laboral (escala 1-4)
`Performance_Rating`	Calificación de desempeño
`Training_Hours_Last_Year`	Horas de capacitación el último año
`Overtime`	Si hace horas extra (1 = Sí, 0 = No)
`Project_Count`	Cantidad de proyectos asignados
`Average_Hours_Worked_Per_Week`	Promedio de horas trabajadas por semana
`Absenteeism`	Días de ausentismo
`Work_Environment_Satisfaction`	Satisfacción con el ambiente laboral
`Relationship_with_Manager`	Calidad de relación con su jefe (escala)
`Job_Involvement`	Nivel de involucramiento en el trabajo
`Distance_From_Home`	Distancia de su casa al trabajo
`Number_of_Companies_Worked`	Cantidad de empresas donde trabajó antes
`Attrition`	Si el empleado dejó la empresa — 1 = Sí, 0 = No (variable objetivo)
---
🔍 Análisis (10 preguntas)
1. ¿Qué porcentaje de empleados ha dejado la empresa?
```sql
SELECT
    CASE WHEN Attrition = 1 THEN 'Sí' ELSE 'No' END AS Se_Fue,
    COUNT(*) AS Total_Empleados,
    CAST(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER() AS DECIMAL(5,2)) AS Porcentaje
FROM Empleados_Attrition
GROUP BY Attrition;
```
Hallazgo: El 19.97% de los empleados (1,997 de 10,000) dejó la empresa, mientras que el 80.03% permanece. Esta tasa es más alta que el promedio "saludable" de la industria (10-15%), lo que indica un problema de retención a nivel general.
2. ¿Qué departamento tiene la mayor rotación de personal?
```sql
SELECT
    Department,
    COUNT(*) AS Total_Empleados,
    SUM(CASE WHEN Attrition = 1 THEN 1 ELSE 0 END) AS Se_Fueron,
    CAST(SUM(CASE WHEN Attrition = 1 THEN 1 ELSE 0 END) * 100.0 / COUNT(*) AS DECIMAL(5,2)) AS Tasa_Rotacion
FROM Empleados_Attrition
GROUP BY Department
ORDER BY Tasa_Rotacion DESC;
```
Hallazgo: Finance tiene la rotación más alta (20.85%), seguido de IT (20.35%). Marketing es el más estable (19.36%). Las diferencias entre departamentos son pequeñas (todas entre 19-21%), lo que sugiere que la rotación es un problema generalizado en toda la empresa, no aislado a un área específica.
3. ¿El ingreso promedio varía según el departamento y el nivel de puesto?
```sql
SELECT
    Department,
    Job_Level,
    AVG(Monthly_Income) AS Ingreso_Promedio
FROM Empleados_Attrition
GROUP BY Department, Job_Level
ORDER BY Department, Job_Level;
```
Hallazgo: El ingreso promedio se mantiene parejo entre departamentos y niveles (entre ~11,000 y ~12,000), sin una relación clara con el nivel jerárquico (Job_Level). Se esperaría que a mayor nivel, mayor ingreso, pero ese patrón no es consistente — por ejemplo, en Finance el nivel 3 gana más que el nivel 4.
4. ¿Trabajar horas extra aumenta la probabilidad de renunciar?
```sql
SELECT
    Overtime,
    COUNT(*) AS Total,
    SUM(CASE WHEN Attrition = 1 THEN 1 ELSE 0 END) AS Se_Fueron,
    CAST(SUM(CASE WHEN Attrition = 1 THEN 1 ELSE 0 END) * 100.0 / COUNT(*) AS DECIMAL(5,2)) AS Tasa_Rotacion
FROM Empleados_Attrition
GROUP BY Overtime;
```
Hallazgo: La rotación es casi idéntica entre quienes hacen horas extra (20.07%) y quienes no (19.87%). La diferencia es mínima, por lo que las horas extra no parecen ser un factor determinante de la rotación en esta empresa.
5. ¿Los empleados que se van tienen menos antigüedad que los que se quedan?
```sql
SELECT
    CASE WHEN Attrition = 1 THEN 'Sí' ELSE 'No' END AS Se_Fue,
    AVG(CAST(Years_at_Company AS DECIMAL(5,2))) AS Antiguedad_Promedio,
    AVG(CAST(Years_in_Current_Role AS DECIMAL(5,2))) AS Anios_en_Rol_Promedio
FROM Empleados_Attrition
GROUP BY Attrition;
```
Hallazgo: No hay diferencia real: los que se van tienen en promedio 14.98 años de antigüedad y los que se quedan 14.92 años — prácticamente iguales. Esto va en contra de la idea común de que "los empleados nuevos son los que más renuncian"; aquí la antigüedad no explica quién se va.
6. ¿Qué puesto de trabajo tiene la menor satisfacción laboral?
```sql
SELECT
    Job_Role,
    AVG(CAST(Job_Satisfaction AS DECIMAL(5,2))) AS Satisfaccion_Promedio,
    COUNT(*) AS Total_Empleados
FROM Empleados_Attrition
GROUP BY Job_Role
HAVING COUNT(*) > 10
ORDER BY Satisfaccion_Promedio ASC;
```
Hallazgo: Manager tiene la satisfacción promedio más baja (3.02 de una escala de 1-4), seguido muy de cerca por Analyst (3.02) y Assistant (3.04). Executive es el más satisfecho (3.07). Las diferencias son pequeñas, pero el rol de Manager merece atención — suelen tener más presión y responsabilidad sin una satisfacción proporcional.
7. ¿Qué departamento tiene el mejor desempeño promedio?
```sql
WITH DesempenoPorDepto AS (
    SELECT
        Department,
        AVG(CAST(Performance_Rating AS DECIMAL(5,2))) AS Desempeno_Promedio
    FROM Empleados_Attrition
    GROUP BY Department
)
SELECT
    Department,
    Desempeno_Promedio,
    RANK() OVER (ORDER BY Desempeno_Promedio DESC) AS Ranking
FROM DesempenoPorDepto
ORDER BY Ranking;
```
Hallazgo: IT tiene el mejor desempeño promedio (2.55), seguido de Marketing (2.52) y HR (2.50). Sales tiene el desempeño más bajo (2.48). Curiosamente, IT también es el segundo departamento con más rotación (pregunta 2) — vale la pena investigar si la empresa está perdiendo a su gente de mejor desempeño en esa área.
8. ¿El balance entre vida personal y trabajo influye en la rotación?
```sql
SELECT
    Work_Life_Balance,
    COUNT(*) AS Total,
    SUM(CASE WHEN Attrition = 1 THEN 1 ELSE 0 END) AS Se_Fueron,
    CAST(SUM(CASE WHEN Attrition = 1 THEN 1 ELSE 0 END) * 100.0 / COUNT(*) AS DECIMAL(5,2)) AS Tasa_Rotacion
FROM Empleados_Attrition
GROUP BY Work_Life_Balance
ORDER BY Work_Life_Balance;
```
Hallazgo: No hay una tendencia clara: la rotación va de 18.82% (nivel 3) a 21.52% (nivel 4). El nivel más alto de balance (4) tiene, sorprendentemente, la mayor rotación — lo contrario a lo esperado. Esto sugiere que el balance vida-trabajo no es, por sí solo, el principal motivo de salida.
9. ¿Qué grupo (por estado civil y género) tiene más ausentismo?
```sql
SELECT
    Marital_Status,
    Gender,
    AVG(CAST(Absenteeism AS DECIMAL(5,2))) AS Ausentismo_Promedio
FROM Empleados_Attrition
GROUP BY Marital_Status, Gender
ORDER BY Ausentismo_Promedio DESC;
```
Hallazgo: Los hombres divorciados tienen el mayor ausentismo promedio (9.59 días), mientras que las mujeres casadas tienen el menor (9.22 días). La diferencia entre el grupo más alto y el más bajo es de menos de medio día, así que el efecto es real pero pequeño — no es un factor determinante por sí solo.
10. ¿Cuántos empleados están en riesgo de renunciar por falta de ascensos y baja satisfacción?
```sql
SELECT COUNT(*) AS Empleados_En_Riesgo
FROM Empleados_Attrition
WHERE Years_Since_Last_Promotion > 5
  AND Job_Satisfaction <= 2
  AND Attrition = 0;

SELECT TOP 10
    Employee_ID, Department, Job_Role, Years_Since_Last_Promotion,
    Job_Satisfaction, Performance_Rating
FROM Empleados_Attrition
WHERE Years_Since_Last_Promotion > 5
  AND Job_Satisfaction <= 2
  AND Attrition = 0
ORDER BY Years_Since_Last_Promotion DESC;
```
Hallazgo: 1,285 empleados activos (12.85% de toda la plantilla) llevan más de 5 años sin ascenso y reportan baja satisfacción laboral — son candidatos de alto riesgo de renuncia. Ejemplos destacados: empleados de IT, Sales, HR y Marketing con 9 años sin ascenso, algunos con desempeño alto (Performance_Rating 3-4) que la empresa no puede permitirse perder.
---
✅ Conclusiones del Análisis
La empresa tiene una tasa de rotación general del 19.97%, por encima del rango saludable de la industria (10-15%), confirmando un problema de retención real.
La rotación no está concentrada en un solo departamento (19-21% en todos), lo que apunta a una causa estructural (cultura, compensación, crecimiento) más que a un problema aislado de un área.
Los factores "clásicos" —horas extra, antigüedad, balance vida-trabajo— no muestran una relación fuerte con la rotación en este análisis. Esto sugiere que la fuga de personal responde a otras causas no tan evidentes a simple vista (compensación relativa, falta de crecimiento, clima laboral específico del equipo).
El ingreso no escala claramente con el nivel de puesto, lo que podría generar frustración e incentivar la salida de empleados senior que no ven una recompensa proporcional a su experiencia.
Hay un grupo grande y concreto (1,285 empleados, 12.85% de la plantilla) estancado hace más de 5 años sin ascenso y con baja satisfacción — este es el segmento más accionable y urgente para retener.
Recomendaciones para RR.HH.:
Revisar la política salarial y de crecimiento a nivel de toda la empresa, no solo por departamento, ya que la rotación es pareja en todas las áreas.
Priorizar el seguimiento de los 1,285 empleados en riesgo (sin ascenso hace +5 años, baja satisfacción) con planes de desarrollo o conversaciones de carrera — es el grupo más fácil de identificar y accionar de inmediato.
Investigar el caso de IT: tiene el mejor desempeño de toda la empresa pero también una de las rotaciones más altas — riesgo de estar perdiendo a los mejores talentos.
Prestar atención especial a los Managers, que muestran la satisfacción laboral más baja entre los roles — podría deberse a exceso de carga o falta de apoyo en su posición intermedia.
Profundizar el análisis con encuestas cualitativas, ya que los factores cuantitativos disponibles (horas extra, balance vida-trabajo) no explican por sí solos la decisión de renunciar.
---
🛠️ Herramientas utilizadas
SQL Server 2019 / SQL Server Management Studio
Dataset: Kaggle — Employee Attrition Prediction Dataset
