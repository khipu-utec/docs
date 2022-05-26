---
title: "Monitorear Recursos"
date: 2022-03-18T10:57:13-05:00
weight: 42
---

{{< toc >}}
## Recomendación General 💡

Asegúrese de reservar la cantidad de RAM y CPUs necesarios para la ejecución de su **job**. No reserve recursos que no necesita, ya que de hacerlo, perjudicará la ejecución de los demás usuarios del cluster. 

A continuación, se muestran algunos ejemplos de como medir el uso de CPU y RAM de su job a fin de que pueda refinar la reserva de recursos.  No olvide revisar la documentación sobre el [envío de jobs](/guia_de_usuario/jobs_schedulling/enviar_jobs/) a Slurm.

### Jobs en ejecución

Si su **job** se encuentra en ejecución, usted puede revisar su uso actual de recursos. Sin embargo, deberá esperar hasta su finalización para ver el uso máximo de recursos durante toda su ejecución. 

La manera más sencilla de revisar el uso instantáneo de recursos es hacer `ssh` al nodo de computación donde su *job* se encuentra ejecutándose. Para saber a que nodo debe hacer `ssh`, ejecute:

```
[juan.perez@khipu ~]$ squeue --me
JOBID PARTITION     NAME     USER  ST       TIME  NODES NODELIST(REASON)
21615 investiga    bert-sar juan   PD       0:00      1 n001
```

Notamos que nuestro job `bert-sar` se encuentra ejecutándose en el valor de la columna `NODELIST` y nos conectamos usando `ssh`.

```
[juan.perez@khipu ~]$ ssh n001
```

Una vez dentro del nodo de cómputo, ejecutaremos `ps` o `htop`.

- `ps` le brindará la información instantánea del uso de recursos cada vez que ejecute el comando. 

    ```
    [juan.perezo@n004 ~]$ ps -u$USER -o %cpu,rss,args
    %CPU   RSS COMMAND
    0.0  2376 python triangle.py
    0.0  2380 python triangle.py
    0.0  2380 python triangle.py
    0.0  2380 python triangle.py
    0.0  2380 python triangle.py
    0.0  2380 python triangle.py
    0.0  2380 python triangle.py
    0.0  2380 python triangle.py
    0.0  2380 python triangle.py
    ```

    El reporte de memoria de `ps` se muestra en KB, podemos notar que los procesos listados consumen alrededor de 2000 KB de RAM y que el uso de los CPUs es casi nulo.

- `htop` se ejecuta de manera interactiva y muestra las estadísticas de uso en vivo. Puede presionar la tecla `u`, ingresar su nombre de usuario y luego `enter` para filtrar solo sus procesos. La información del uso de memoria, se encuentra en la columna RES. Para solicitar ayuda puede presionar `?` y si desea salir `q` .

<p align="center">
  <img src="/guia_de_usuario/jobs_schedulling/htop.png">
</p>


### Jobs finalizados

Slurm guarda las estadísticas de cada job, incluído cuanta memoria y CPU fue utilizada.

#### seff

- Luego de finalizado un job, puede ejecutar `seff<jobid>` para obtener información útil sobre la ejecución de su job, tal como la memoria usada y a que porcentaje de la memoria reserva corresponde.

```
[juan.perez@khipu ~]$ seff 21886
Job ID: 21886
Cluster: cluster
Use of uninitialized value $group in concatenation (.) or string at /usr/bin/seff line 154, <DATA> line 602.
User/Group: juan.perez/
State: COMPLETED (exit code 0)
Cores: 1
CPU Utilized: 00:00:00
CPU Efficiency: 0.00% of 00:00:00 core-walltime
Job Wall-clock time: 00:00:00
Memory Utilized: 0.00 MB (estimated maximum)
Memory Efficiency: 0.00% of 50.00 GB (50.00 GB/core)
```

#### sacct

También se puede usar `sacct` para obtener la información del job. Lamentablemente, el output por defecto de `sacct` no es del todo entendible, por ello se recomienda procesar la salida de la siguiente manera. 

```
[juan.perez@khipu ~]$ export SACCT_FORMAT="JobID%20,JobName,User,Partition,NodeList,Elapsed,State,ExitCode,MaxRSS,AllocTRES%32"
[juan.perez@khipu ~]$ sacct -j 21886
JobID    JobName   User  Partition  NodeList    Elapsed   State ExitCode     MaxRSS     AllocTRES
-------------------- ---------- --------- ---------- --------------- ---------- ---------- -------- ----------
21886 125000000+ juan.pe+ investiga+   g001   00:00:00  COMPLETED      0:0       billing=1,cpu=1,mem=50G,node=1
21886.batch      batch                  g001   00:00:00  COMPLETED      0:0          cpu=1,mem=50G,node=1
```