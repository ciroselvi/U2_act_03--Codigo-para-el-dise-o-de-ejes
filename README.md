#Esta primera celdad trata de la introduccion de datos de forma manual para que a partir de aqui se hagan los calculos necesarios
#Ojo este codigo fue hecho con jupiter o .ipnyb, asi que tomen cada parte que este separada como si fueran celdas independientes
#La primera seccion tiene como funcion calcular los valores de q, qc, kt, kts; que son valores necesarios para calcular despues la concentracion de esfuerzos

# Ingreso manual de variables principales
print('--- Ingreso de variables ---')
d = float(input('Diametro menor d [pulgadas]: '))
D = float(input('Diametro mayor D [pulgadas]: '))
r = float(input('Radio de muesca r [pulgadas]: '))
Ma = float(input('Momento alternante Ma [lb-in]: '))
Mm = float(input('Momento medio Mm [lb-in]: '))
Ta = float(input('Torque alternante Ta [lb-in]: '))
Tm = float(input('Torque medio Tm [lb-in]: '))
sy = float(input('Resistencia a la fluencia Sy [psi]: '))
Sut = float(input('Resistencia última Sut [psi]: '))
Se = float(input('Límite de fatiga corregido Se [psi]: '))
n = float(input('Factor de seguridad n: '))
c= d/2;
# Cálculo automático de factores usando funciones y datos geométricos/materiales
import math as mt
r_mm = r * 25.4
rd = r/d
Dd_kt = D/d
Dd_kts = D/d
def curva_q(r_mm, Sut):
    if Sut == 200:
        return 0.95 - 0.05 * mt.exp(-r_mm/0.5)
    elif Sut == 150:
        return 0.90 - 0.10 * mt.exp(-r_mm/0.4)
    elif Sut == 100:
        return 0.85 - 0.15 * mt.exp(-r_mm/0.3)
    elif Sut == 60:
        return 0.75 - 0.20 * mt.exp(-r_mm/0.2)
    else:
        if Sut < 60: Sut = 60
        if Sut > 200: Sut = 200
        q_60 = curva_q(r_mm, 60)
        q_100 = curva_q(r_mm, 100)
        q_150 = curva_q(r_mm, 150)
        q_200 = curva_q(r_mm, 200)
        if Sut <= 100:
            return q_60 + (q_100 - q_60) * (Sut-60)/(100-60)
        elif Sut <= 150:
            return q_100 + (q_150 - q_100) * (Sut-100)/(150-100)
        else:
            return q_150 + (q_200 - q_150) * (Sut-150)/(200-150)

def curva_qc(r_mm, Sut):
    if Sut == 200:
        return 0.98 - 0.08 * mt.exp(-r_mm/0.5)
    elif Sut == 150:
        return 0.93 - 0.13 * mt.exp(-r_mm/0.4)
    elif Sut == 100:
        return 0.88 - 0.18 * mt.exp(-r_mm/0.3)
    elif Sut == 60:
        return 0.78 - 0.22 * mt.exp(-r_mm/0.2)
    else:
        if Sut < 60: Sut = 60
        if Sut > 200: Sut = 200
        qc_60 = curva_qc(r_mm, 60)
        qc_100 = curva_qc(r_mm, 100)
        qc_150 = curva_qc(r_mm, 150)
        qc_200 = curva_qc(r_mm, 200)
        if Sut <= 100:
            return qc_60 + (qc_100 - qc_60) * (Sut-60)/(100-60)
        elif Sut <= 150:
            return qc_100 + (qc_150 - qc_100) * (Sut-100)/(150-100)
        else:
            return qc_150 + (qc_200 - qc_150) * (Sut-150)/(200-150)

def curva_kt(rd, Dd):
    if Dd == 3:
        return 1.4 + 1.6 * mt.exp(-rd/0.07)
    elif Dd == 1.5:
        return 1.2 + 1.48 * mt.exp(-rd/0.09)
    elif Dd == 1.1:
        return 1.1 + 1.35 * mt.exp(-rd/0.11)
    elif Dd == 1.02:
        return 1.05 + 1.25 * mt.exp(-rd/0.13)
    else:
        if Dd < 1.02: Dd = 1.02
        if Dd > 3: Dd = 3
        kt_102 = curva_kt(rd, 1.02)
        kt_11 = curva_kt(rd, 1.1)
        kt_15 = curva_kt(rd, 1.5)
        kt_3 = curva_kt(rd, 3)
        if Dd <= 1.1:
            return kt_102 + (kt_11 - kt_102) * (Dd-1.02)/(1.1-1.02)
        elif Dd <= 1.5:
            return kt_11 + (kt_15 - kt_11) * (Dd-1.1)/(1.5-1.1)
        else:
            return kt_15 + (kt_3 - kt_15) * (Dd-1.5)/(3-1.5)

def curva_kts(rd, Dd):
    if Dd == 2:
        return 1.2 + 1.6 * mt.exp(-rd/0.07)
    elif Dd == 1.33:
        return 1.1 + 1.48 * mt.exp(-rd/0.09)
    elif Dd == 1.20:
        return 1.05 + 1.35 * mt.exp(-rd/0.11)
    elif Dd == 1.09:
        return 1.02 + 1.25 * mt.exp(-rd/0.13)
    else:
        if Dd < 1.09: Dd = 1.09
        if Dd > 2: Dd = 2
        kts_109 = curva_kts(rd, 1.09)
        kts_120 = curva_kts(rd, 1.20)
        kts_133 = curva_kts(rd, 1.33)
        kts_2 = curva_kts(rd, 2)
        if Dd <= 1.20:
            return kts_109 + (kts_120 - kts_109) * (Dd-1.09)/(1.20-1.09)
        elif Dd <= 1.33:
            return kts_120 + (kts_133 - kts_120) * (Dd-1.20)/(1.33-1.20)
        else:
            return kts_133 + (kts_2 - kts_133) * (Dd-1.33)/(2-1.33)

# Asignación automática de los valores
q = curva_q(r_mm, Sut)
qc = curva_qc(r_mm, Sut)
kt = curva_kt(rd, Dd_kt)
kts = curva_kts(rd, Dd_kts)
print(f'\nq calculado: {q:.3f}')


print(f'qc calculado: {qc:.3f}')
print(f'Kt calculado: {kt:.3f}')
print(f'Kts calculado: {kts:.3f}')

