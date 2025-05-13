#linkedin #oracle #docker

🧠 ¿Querés probar Oracle 23ai (la última versión)... gratis y en menos de 2 minutos?  
  
Sí, se puede.  
  
Con Docker Desktop lo tenés corriendo en tu máquina con 1 sola línea:  
  
docker run -d -p 1521:1521 -e ORACLE_PASSWORD=SysPassword1 gvenzl/oracle-free  
  
👉 Sin instalar nada raro  
👉 Sin licencias  
👉 Sin perder tiempo  
  
🔥 ¿Y cómo accedés a la base?  
  
Desde Docker Desktop:  
  
Entrás a la consola del contenedor y ejecutas:  
  
sqlplus / as sysdba  
  
👀 ¿Querés la string de conexión para SQL Developer, Python o cualquier cliente externo?  
  
También está ahí mismo:  
  
cat $ORACLE_HOME/network/admin/tnsnames.ora  
  
Copiás los datos… y listo.  
  
🔹 Oracle 23ai corriendo local  
🔹 Incluye vector search  
🔹 Ideal para labs de AI, pruebas y aprendizaje sin romper nada  
  
Y sí → es una imagen oficial mantenida por @Gerald Venzl (Oracle)

![[Pasted image 20250508131110.png]]