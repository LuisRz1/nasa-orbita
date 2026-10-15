---
id: guardado
type: especificacion
status: especificado-sin-implementar
updated: 2026-10-14
tags: [orbita, persistencia]
---
# Guardado y recuperación

IndexedDB mediante Dexie será el almacenamiento principal. El usuario podrá exportar e importar una partida como archivo de datos validado.

## Contenido persistente

Versión de esquema y contenido; perfil local; progreso de misiones; recursos estables; IDs de hallazgos; mejoras; semillas necesarias; memoria y decisiones escritas de LUMA; opciones.

No guardar referencias de objetos Phaser, nodos DOM, funciones, credenciales ni una dependencia del historial completo de chat.

## Escritura

Guardar al completar hitos y al regresar, con escrituras transaccionales. El progreso visible confirmado debe corresponder a una escritura recuperable.

Al ocultar la pestaña, pausar la simulación y solicitar guardado cuando sea posible. No depender únicamente del cierre de la pestaña para persistir.

## Lectura y migración

Validar antes de reemplazar la partida activa. Mantener respaldo antes de migrar y rechazar versiones futuras con explicación, sin sobrescribir.

Si la cuota de almacenamiento o permisos impiden escribir, informar y ofrecer exportación. Una importación dañada no borra la partida existente.

## Aceptación

Recargar en un hito, repetir una entrega y exportar/importar conserva el estado sin premios duplicados. Sin red se puede completar la región ya cargada; una región no descargada debe comunicar su indisponibilidad.
