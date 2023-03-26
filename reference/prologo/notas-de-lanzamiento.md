# Notas de Lanzamiento

## Esquema de Versionamiento

Laravel y sus otros paquetes siguen el [Versionado Semántico](https://semver.org/). Las versiones mayores del framework se publican cada año (\~Q1), mientras que las versiones menores y de parche pueden publicarse cada semana. Las versiones menores y de parche nunca deben contener cambios de última hora.

Cuando hagas referencia al framework Laravel o a sus componentes desde tu aplicación o paquete, deberías usar siempre una restricción de versión como `^10.0`, ya que las versiones mayores de Laravel incluyen cambios de última hora. Sin embargo, nos esforzamos para asegurar que siempre se puede actualizar a una nueva versión principal en un día o menos.

### Argumentos con Nombre

Los argumentos con nombre no están cubiertos por las directrices de compatibilidad con versiones anteriores de Laravel. Podemos optar por cambiar el nombre de los argumentos de función cuando sea necesario con el fin de mejorar la base de código Laravel. Por lo tanto, el uso de argumentos con nombre al llamar a métodos Laravel debe hacerse con cautela y con la comprensión de que los nombres de los parámetros pueden cambiar en el futuro.

## Políticas de Soporte

Para todas las versiones de Laravel, se proporcionan correcciones de errores durante 18 meses y correcciones de seguridad durante 2 años. Para todas las bibliotecas adicionales, incluyendo Lumen, sólo la última versión principal recibe correcciones de errores. Además, por favor revise las versiones de bases de datos soportadas por Laravel.

| Versión | PHP (\*)  | Lanzamiento             | Corrección de errores hasta | Corrección de seguridad hasta |
| ------- | --------- | ----------------------- | --------------------------- | :---------------------------: |
| 8       | 7.3 - 8.1 | 8 de septiembre de 2020 | 26 de julio de 2022         |      24 de enero de 2023      |
| 9       | 8.0 - 8.2 | 8 de febrero de 2022    | 8 de agosto de 2023         |      6 de febrero de 2024     |
| 10      | 8.1 - 8.2 | 14 de febrero de 2023   | 6 de agosto de 2024         |      4 de febrero de 2025     |
| 11      | 8.2       | T1 2024                 | 5 de agosto de 2025         |      3 de febrero de 2026     |
