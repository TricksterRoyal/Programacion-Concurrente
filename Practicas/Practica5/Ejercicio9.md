# Ejercicio 9

Una empresa de limpieza se encarga de recolectar residuos en una ciudad. Hay P personas que continuamente hacen reclamos para que pasen por su casa. La empresa tiene 2 camiones, que cuando alguno de ellos está libre lo envía a recolectar los residuos de la persona que más reclamos ha hecho.

```ada
TASK TYPE PERSONA IS
  ENTRY SEND_ID(ID_PERSONA: IN INTEGER);
END PERSONA;

TASK EMPRESA IS
  ENTRY RECLAMAR(RECLAMO: IN STRING, ID_PERSONA: IN INTEGER);
  ENTRY TOMAR_RECLAMO(RECLAMO: OUT STRING);
END EMPRESA;

TASK TYPE CAMION;

TASK BODY PERSONA IS
  RECLAMO: STRING; ID: INTEGER;
  ACCEPT SEND_ID(ID_PERSONA: IN INTEGER) DO
    ID := ID_PERSONA;
  END SEND_ID;
  WHILE(TRUE)LOOP
    RECLAMO := GENERAR_RECLAMO();
    EMPRESA.RECLAMAR(RECLAMO, ID);
  END LOOP;
END PERSONA;

TASK BODY CAMION IS
  RECLAMO: STRING;
  WHILE(TRUE)LOOP
    EMPRESA.TOMAR_RECLAMO(RECLAMO);
    IF(RECLAMO != -1) THEN
      --RESOLVER RECLAMO
    END IF;
  END LOOP;
END CAMION;

TASK BODY EMPRESA
  COLAS: ARRAY(1..P) OF QUEUE;
  CONTADORES: ARRAY(1..P) OF INTEGER;
  I, CUR_ID: INTEGER; MAX: INTEGER := -1;
  WHILE(TRUE)LOOP
    SELECT
      ACCEPT RECLAMAR(RECLAMO: IN STRING, ID_PERSONA: IN INTEGER) IS
        COLA[ID_PERSONA].PUSH(RECLAMO);
        CONTADORES[ID_PERSONA] := CONTADORES[ID_PERSONA] + 1;
      END RECLAMAR;
    OR
      ACCEPT TOMAR_RECLAMO(RECLAMO: OUT STRING) IS
        FOR I TO 1..P LOOP
          IF(CONTADORES[I] > MAX) THEN
            MAX := CONTADORES[I];
            CUR_ID := I;
          END IF;
          IF(!COLAS[I].EMPTY()) THEN
            RECLAMO := COLA[I].POP();
          ELSE
            RECLAMO := -1;
          END IF;
        END LOOP
        RECLAMO := COLA.POP();
      END TOMAR_RECLAMO;
    END SELECT;
  END LOOP;
END EMPRESA;

VAR
  PERSONAS := ARRAY(1..P) OF PERSONA;
  CAMIONES := ARRAY(1..2) OF CAMION;
  I: INTEGER;
BEGIN
  FOR I TO 1..P LOOP
    PERSONAS[I].SEND_ID(I);
  END LOOP;
END
```