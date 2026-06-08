# ~~Caso de Uso: Descargar Perfil de Dispositivo~~ — FUSIONADO

**Código:** CU-26
**Archivo:** CU-26-proveer-perfil-motor_v2.0.md
**Versión:** 2.0
**Estado:** ⚠️ FUSIONADO con CU-21 v2.0
**Fecha:** 2026-05-27
**Autor:** Equipo Funcional / Arquitectura
**Glosario:** [docs/03_ux-ui/glosario_v1.0.md](../../03_ux-ui/glosario_v1.0.md)

> **Nota terminológica (2026-04-25):** este CU usaba originalmente "perfil de impresora". Se actualizó la prosa al término canónico **"perfil de dispositivo"** según el glosario.

---

# AVISO — CASO DE USO FUSIONADO

> **Este caso de uso ha sido absorbido por CU-21 v2.0 (Cargar Perfil de Dispositivo).**

## Motivo de la fusión

En la versión 1.0, CU-26 describía la descarga de perfiles de dispositivo desde una API REST como un flujo independiente. Con la adopción del patrón de **proveedor inyectable** (`IDeviceProfileProvider`) en CU-21 v2.0, la descarga remota se convierte en una posible implementación del proveedor, no en un caso de uso separado.

## Cómo se cubre ahora

* **CU-21 v2.0** define el contrato `IDeviceProfileProvider`.
* La librería provee `InMemoryDeviceProfileProvider` como implementación default.
* Si el cliente necesita obtener perfiles desde una API REST, implementa `IDeviceProfileProvider` con su propia lógica HTTP:

  ```csharp
  public class ApiDeviceProfileProvider : IDeviceProfileProvider
  {
      private readonly HttpClient _httpClient;

      public ApiDeviceProfileProvider(HttpClient httpClient)
          => _httpClient = httpClient;

      public DeviceProfile? GetProfile(string name)
      {
          // Lógica HTTP del cliente
      }

      public IEnumerable<DeviceProfile> GetAll() { ... }
      public void Add(DeviceProfile profile) { ... }
  }
  ```

* El registro en DI:

  ```csharp
  services.AddSingleton<IDeviceProfileProvider, ApiDeviceProfileProvider>();
  ```

## Referencia

Ver: **CU-21-cargar-perfil-impresora_v2.0.md**

---

# Contenido original v1.0 (conservado como referencia)

El contenido original de este caso de uso describía:

* Descarga de perfil desde API REST
* Construcción de petición HTTP con identificador y versión
* Validación de respuesta HTTP y JSON
* Almacenamiento en memoria/caché
* Manejo de errores: API no disponible, 404, JSON inválido, fallos de red

Todos estos escenarios quedan cubiertos por la implementación personalizada que el cliente realice de `IDeviceProfileProvider`.

---

# Cambios respecto a versión 2.0 (registro)

* **Edición terminológica (2026-04-25):** "perfil de impresora" → "perfil de dispositivo" en prosa, alineado con [docs/03_ux-ui/glosario_v1.0.md](../../03_ux-ui/glosario_v1.0.md). El nombre del archivo se conserva por estabilidad de links.

---

**Fin del documento**
