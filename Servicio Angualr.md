# ServiceWebSocketAngular
# 

import { Injectable } from '@angular/core';

import { JwtService } from './jwt.service';

import { AlojamientoService } from './alojamiento.service';

import { HabitacionService } from './habitacion.service';

import { ip } from '../models/api.model';

import { UsuarioService } from './usuario.service';

import { ClienteService } from './cliente.service';

import { ReservaService } from './reserva.service';

import Swal from 'sweetalert2';

@Injectable({

  providedIn: 'root',
  
})

export class WebsocketService {

  socket: WebSocket | undefined;
  
  constructor(
  
    private jwt: JwtService,
    private Aloj$: AlojamientoService,
    private Habit$: HabitacionService,
    private User$: UsuarioService,
    private Client$: ClienteService,
    private Reserv$: ReservaService
    
  ) {}

``` declaro una propiedad para definir el numero de veces que se debe probar en caso que la conexion se haya caido una ves establecida```
  private MAX_RECONNECTION = 5;
  
  private contador = 0;

  setsock() {

    declaro una constante para obtener el protocolo que se este usando y asi poder alternar el protocolo que usara Django para establecer
    
    const wsScheme = window.location.protocol == 'https:' ? 'wss' : 'ws';

    luego de saber el tipo de protocolo se procede a establecer la conexion con el backend, 
    en donde ip es la direccion del servicio web de Render y access el encabezado del token
    y es en esta linea donde resulta el error ya que me dice que la conexion fallo
    
    this.socket = new WebSocket(`${wsScheme}://${ip}/ws/?access=${this.jwt.Token}`);
    
    this.socket.onopen = () => {
    
    mediante la consola compruebo que se establecio la conexion con el servidor backend
    y el contador comienza a aumentar en caso que la conexion se haya caido
    
      console.log('WebSockets connection created for Socket Service');
      if (this.contador > 1) {
        Swal.fire({
          position: 'top-end',
          icon: 'success',
          title: 'WebSocket reconectado, si hay multiples usuarios trabajando es recomendable recargar la pagina',
          showConfirmButton: true,
        })
      }
      this.contador = 1;
    };

    this.socket.onmessage = (event: MessageEvent) => {
      let action = JSON.parse(event.data);
      action = {
        data: JSON.parse(action.message.data),
        event: action.message.event,
        model: action.message.model
      }
      switch (action.model) {
        case 'Alojamiento':
          this.Aloj$.updateList(action);
          break;
        case 'Habitacion':
          this.Habit$.updateList(action);
          break;
        case 'Usuario':
          this.User$.updateList(action);
          break;
        case 'Cliente':
          this.Client$.updateList(action);
          break;
        case 'Reserva':
          this.Reserv$.updateList(action);
          break;
      }
    };
    this.socket.onclose = () => {
      if (this.contador !== 0 && this.contador <= this.MAX_RECONNECTION) {
        console.log(
          `reconectando ws intento ${this.contador} de ${this.MAX_RECONNECTION}`
        );
        this.socket = undefined;
        const p3 = new Promise<void>((resolve) => {
          Swal.fire({
            position: 'top-end',
            icon: 'success',
            title: `Reconectando websocket intento ${this.contador} de ${this.MAX_RECONNECTION}`,
            showConfirmButton: true,
          })
          this.contador++;
          setTimeout(() => {
            this.setsock();
          }, 3000 * this.contador);
          resolve();
        });
      } else {
        Swal.fire({
          position: 'top-end',
          icon: 'error',
          title: 'Error de conexion, por favor verificar su conexion a internet o recargar la pagina',
          showConfirmButton: true,
        })

      }
    };
    if (this.socket.readyState === WebSocket.OPEN) {
      this.socket.onopen(null as any);
    }
  }
}
