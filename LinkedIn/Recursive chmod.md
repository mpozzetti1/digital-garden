#linkedin #oracle 

💀 Borré un servidor Linux de la empresa en menos de 1 segundo.  
  
Y solo por este comando:  
  
chmod -R 700 $ORACLE_HOME/ (faltaba el recursive en la foto)  
  
(Spoiler: $ORACLE_HOME no estaba inicializado)  
  
Sin querer, cambié los permisos de todo el sistema.  
  
Sí. Todo.  
  
Root incluido.  
  
¿Lo peor?  
  
No hice ni un solo backup. Ninguno.  
  
Era un servidor no productivo... pero contenía una base de datos Oracle que centralizaba los backups de otras con RMAN.  
  
Esto sucedió cuando era muy Junior, así que entenderás que no fue mi mejor momento. Pero aprendí:  
  
→ Evitar usar variables de ambiente en comandos chmod.  
→ Entender lo delicado que puede ser usar el usuario root en Linux.  
→ Hacer un backup del servidor antes de empezar una tarea importante.  
  
Hoy lo cuento como anécdota y nos reímos.  
  
Pero aquel día no supe que había pasado hasta que un compañero experto en Linux pudo entrar al servidor corrupto y pudo ver el comando que rompió todo.  
  
Cuál fue tu peor desastre en un servidor?  
  
Te leo.