# Workflow de experimentación — resumen

Este documento resume el proceso completo de experimentación, desde la corrida de los modelos hasta la selección del modelo final.

## 1. Corrida de experimentos

Se corrieron varios experimentos, cada uno con las mismas 10 semillas (`468889`, `567793`, `347671`, `678607`, `702787`, `117809`, `925387`, `382961`, `744559`, `777199`), generando un modelo LightGBM por experimento/semilla y subiendo a Kaggle los distintos cortes de predicción de cada corrida.

Notebook utilizada: `z729_final_junior_AUTO.ipynb`

> En los experimentos colaborativos, indagamos sobre "FE Historico". Para esta predicción final, se decidió realizar algunos experimentos más sobre Data drifting.

## 2. Resubmit de archivos faltantes

Al llegar al límite de 100 submits diarios de Kaggle, quedaron varios archivos de predicción ya generados pero sin subir. Se usó el script de resubmit para localizar esos `.csv` ya guardados en el bucker y completar su envío a Kaggle, reconstruyendo el mismo nombre de archivo y el mismo comentario que hubiera generado el submit original (leyendo los hiperparámetros desde `results_summary_<experimento>.txt`).

Notebook utilizada: `aux_kaggle_resubmit.ipynb`

## 3. Comparación de modelos con test de Wilcoxon

Para cada experimento se tomó, por cada una de las 10 semillas, la mejor ganancia obtenida entre los cortes subidos a Kaggle. Con esos 10 valores por experimento se calcularon promedio, máximo y desvío estándar, y se corrió un test de Wilcoxon pareado entre cada par de experimentos (mismo orden de semillas en ambos vectores) para determinar si había diferencias estadísticamente significativas entre ellos.

Notebook utilizada: `compare_models.ipynb`

### Ganancias por semilla y experimento

| experimento / semilla | Data drifting - estandarizar con bug | Data drifting - deflación (baseline) | Data drifting - dólar blue | data drifting - estandarizar sin bug | data drifting - ninguno (92023) |
|---|---|---|---|---|---|
| 468889 | 90965 | 80830 | 86978 | 87809 | 87975 |
| 567793 | 92627 | 86230 | 83821 | 87559 | 85566 |
| 347671 | 91381 | 87310 | 83406 | 87809 | 88390 |
| 678607 | 93042 | 84818 | 84070 | 90716 | 87975 |
| 702787 | 88889 | 90716 | 83073 | 90633 | 85399 |
| 117809 | 93541 | 83738 | 84569 | 91298 | 91796 |
| 925387 | 89470 | 81412 | 89387 | 90550 | 83987 |
| 382961 | 88473 | 92128 | 87559 | 89055 | 81744 |
| 744559 | 89802 | 87310 | 82243 | 86147 | 86812 |
| 777199 | 91879 | 91796 | 83406 | 83904 | 83406 |
| **avg** | **91006,9** | **86628,8** | **84851,2** | **88548** | **86305** |
| **max** | **93541** | **92128** | **89387** | **91298** | **91796** |
| **sd** | **1789** | **4044** | **2317** | **2364** | **2911** |

### Resultados del test de Wilcoxon (p-values)

| Resultados wilcox.test -> p-values | Data drifting - estandarizar bug | Data drifting - deflación (baseline) | Data drifting - dólar blue | data drifting - estandarizar sin bug | data drifting - ninguno |
|---|---|---|---|---|---|
| **Data drifting - estandarizar bug** | — | 0,0273 | 0,0020 | 0,0273 | 0,0020 |
| **Data drifting - deflación (baseline)** | 0,0273 | — | 0,3750 | 0,3750 | 1,0000 |
| **Data drifting - dólar blue** | 0,0020 | 0,3750 | — | 0,0020 | 0,3477 |
| **data drifting - estandarizar sin bug** | 0,0273 | 0,3750 | 0,0020 | — | 0,1387 |
| **data drifting - ninguno** | 0,0020 | 1,0000 | 0,3477 | 0,1387 | — |

**Lectura:** *Data drifting - estandarizar con bug* tuvo el mayor promedio de ganancia (91006,9) y, además, es el único experimento con diferencias estadísticamente significativas (p < 0.05) frente a **todos** los demás. El resto de los experimentos no muestran diferencias significativas entre sí (p > 0.05 en casi todos los pares), por lo que se lo consideró el mejor modelo.

## 4. Ensamble final y predicción

Con el experimento ganador (*Data drifting - estandarizar con bug*), se promediaron las probabilidades generadas por los modelos de las 10 semillas (ensamble), generando una única predicción combinada. Sobre esa predicción promediada se subieron nuevamente los distintos cortes a Kaggle, y se eligió como predicción final la subida con mejor resultado entre esos cortes.

Notebook utilizada: `final_ensemble.ipynb`