```python
from mesa import Agent, Model
from mesa.time import SimultaneousActivation
from mesa.time import StagedActivation
from mesa.space import MultiGrid
import random
import string
import time

arr_Nombres = ["Juan","Pepe","Ulises","Jorge","Kevin","Maria","Elizabeth","Paulina","Ximena","Jose"]
arr_Tipos = ["Estandar","Express","Siguiente dia"]
caracters = "0123456789"
arr_folio = list(caracters)

#Conceptos
#Concepto paquete
class paquete():
    clave = 'CLAVE'
    destinatario = 'DESTINATARIO'
    telefono = 'TELEFONO'
    direccion = 'DIRECCION'
    
    def __init__(self,clave,nombre,tel,direc):
        self.clave = clave
        self.destinatario = nombre
        self.telefono = tel
        self.direccion = direc
        
#Concepto envio
class envio():
    tipo = 'TIPO'
    precio = 'PRECIO'
    
    def __init__(self,t,p):
        self.tipo = t
        self.precio = p

#Agentes
#Agente usuario
class Usuario(Agent): 
    def __init__(self, unique_id, model):
        super().__init__(unique_id, model)
        env = envio("TIPO","PRECIO")
        self.objeto = env
    #Predicado solicitar envio
    def solicitarEnvio(self):
        self.objeto.tipo = input("Ingresa el tipo de envio: ")

    #Accion pagar envio
    def pagarEnvio(self):
        pago = int(input("Ingresa el pago: "))
        deuda = int(self.objeto.precio)
        if(deuda-pago <= 0):
            print("Gracias por tu pago cambio : "+str((deuda-pago)*-1))
        else:
            print("No se pudo completar el pago necesitas "+str(deuda-pago)+" pesos mas")
            self.objeto.precio = str(deuda-pago)
            self.pagarEnvio()

    #Setup
    def step(self):
        print("Buen dia necesito un envio")
        self.solicitarEnvio()
        print("{},{} activated".format(self.unique_id,self.objeto.tipo))

#Agente empleado
class Empleado(Agent):
    def __init__(self, unique_id, model, agent):
        super().__init__(unique_id, model)
        a = "FOLIO"
        b = str(arr_Nombres[random.randint(0,len(arr_Nombres)-1)])
        pkg = paquete(a,b,"00000","0000")
        self.objeto = pkg
        self.agent = agent
        
    #Predicado pedir datos 
    def pedirDatos(self):
        self.objeto.clave = generaFolio()
        self.objeto.destinatario = input("Ingresa nombre del destino: ")
        self.objeto.telefono = input("Ingresa el telefono de referencia: ")
        self.objeto.direccion = input("Ingresa la direccion de destino: ")

    #Accion cobrar envio   
    def cobrarEnvio(self,agent):
        if(agent.objeto.tipo == "EXPRESS"):
            agent.objeto.precio = "200"
            print("Total: 200 Gracias")
        elif(agent.objeto.tipo == "ESTANDAR"):
            agent.objeto.precio = "100"
            print("Total: 100 Gracias")
        else:
            print("No se pudo chavo")
            
    #Setup      
    def step(self):
        print("Hola mucho gusto soy el boot de servicio {} ".format(self.unique_id))
        print("Que tipo de servicio necesita EXPRESS = 200 ESTANDAR = 100")
        self.pedirDatos()
        print("Folio: {} Para: {} Telefono: {} Direccion: {}".format(self.objeto.clave,self.objeto.destinatario,self.objeto.telefono,self.objeto.direccion))
        self.cobrarEnvio(self.agent)
        self.agent.pagarEnvio()
        
#Agente transportista   
class Transportista(Agent):
    def __init__(self, unique_id,model,agent, agent2):
        super().__init__(unique_id,model)
        self.agent = agent
        self.agent2 = agent2
    #Predicado pedir datos    
    def pedirDatos(self):
        self.codR = generaFolio()

    #Setup
    def step(self):
        print("{} Tu paquete ha sido enviado codigo: {} tipo: {}".format(self.unique_id,self.agent2.objeto.clave,self.agent.objeto.tipo))
        if(self.agent.objeto.tipo == "EXPRESS"):
            ran = random.randint(1,2)
        else:
            ran = random.randint(3,7)
            
        print("Tu paquete esta en transito...") 
        pausa(ran)
        print("{} Tu paquete ha sido entregado codigo: {} tipo: {}".format(self.unique_id,self.agent2.objeto.clave,self.agent.objeto.tipo))
        print("Llego en "+str(ran)+" dias")

#Ontología paqueteria
class OntologiaPaqueteria(Model):
    def __init__(self, n_agents):
        super().__init__()
        self.schedule = StagedActivation(self)
        for i in range(n_agents):
            a = Usuario(i, self)
            self.schedule.add(a)
            b = Empleado((i+1)*10,self,a)
            self.schedule.add(b)
            c = Transportista((i+1)*100,self,a,b)
            self.schedule.add(c)  
        
    def step(self):
        self.schedule.step()
        
def pausa(n):
    time.sleep(n)
    
def generaFolio():
    number_of_strings = 5
    length_of_string = 8
    a = ''
    for x in range(number_of_strings):
      a = ''.join(random.choice(string.ascii_letters + string.digits) for _ in range(10))
    return a

#Main        
band = False
while(band == False):
    model = OntologiaPaqueteria(1)
    model.step()
    resp = input("Otro :")
    if(resp == "No" or resp =="n" or resp=="NO" or resp=="N"):
        band = True

```

    Buen dia necesito un envio
    Ingresa el tipo de envio: ESTANDAR
    0,ESTANDAR activated
    Hola mucho gusto soy el boot de servicio 10 
    Que tipo de servicio necesita EXPRESS = 200 ESTANDAR = 100
    Ingresa nombre del destino: JUAN
    Ingresa el telefono de referencia: SA5D5S6A4D
    Ingresa la direccion de destino: JUAREZ NORTE 136
    Folio: y7mYlhGXgB Para: JUAN Telefono: SA5D5S6A4D Direccion: JUAREZ NORTE 136
    Total: 100 Gracias
    Ingresa el pago: 85
    No se pudo completar el pago necesitas 15 pesos mas
    Ingresa el pago: 12
    No se pudo completar el pago necesitas 3 pesos mas
    Ingresa el pago: 3
    Gracias por tu pago cambio : 0
    100 Tu paquete ha sido enviado codigo: y7mYlhGXgB tipo: ESTANDAR
    Tu paquete esta en transito...
    100 Tu paquete ha sido entregado codigo: y7mYlhGXgB tipo: ESTANDAR
    Llego en 7 dias
    Otro :S
    Buen dia necesito un envio
    


```python

```


```python

```


```python

```
