#!/usr/bin/env node
/**
 * Recalcula datos/factor.json a partir del indice UVA.
 *
 *   factor = UVA de hoy / UVA de la fecha base
 *
 * Los precios de index.html quedan congelados a la fecha base y se multiplican
 * por este factor en el navegador. Este script nunca toca index.html.
 *
 * Salidas para el workflow (GITHUB_OUTPUT):
 *   cambio=true|false   hubo variacion respecto del factor anterior
 *   salto=true|false    la variacion supera el umbral y requiere revision humana
 */

import { readFile, writeFile } from 'node:fs/promises';

const FUENTE  = 'https://api.argentinadatos.com/v1/finanzas/indices/uva';
const UMBRAL  = 0.15;  // 15% de salto entre corridas dispara revision manual
const BASE    = 'datos/base.json';
const FACTOR  = 'datos/factor.json';

const leer = async (p) => JSON.parse(await readFile(p, 'utf8'));
const guardar = (p, o) => writeFile(p, JSON.stringify(o, null, 2) + '\n');

const MESES = ['Enero','Febrero','Marzo','Abril','Mayo','Junio',
               'Julio','Agosto','Septiembre','Octubre','Noviembre','Diciembre'];

function etiqueta(fecha) {
  const [a, m] = fecha.split('-');
  return `${MESES[+m - 1]} ${a}`;
}

async function ultimoUVA() {
  const r = await fetch(FUENTE, { headers: { accept: 'application/json' } });
  if (!r.ok) throw new Error(`La fuente respondio ${r.status}`);
  const serie = await r.json();
  if (!Array.isArray(serie) || !serie.length) throw new Error('La serie vino vacia');

  const ultimo = serie[serie.length - 1];
  const valor = Number(ultimo.valor);
  if (!Number.isFinite(valor) || valor <= 0) {
    throw new Error(`Valor UVA invalido: ${ultimo.valor}`);
  }
  return { valor, fecha: ultimo.fecha };
}

async function main() {
  const base = await leer(BASE);
  const previo = await leer(FACTOR).catch(() => ({ factor: 1 }));
  const uva = await ultimoUVA();

  // Primera corrida: la fecha de hoy queda como ancla y el factor arranca en 1.
  if (base.uva_base == null) {
    base.uva_base = uva.valor;
    base.fecha_base = uva.fecha;
    await guardar(BASE, base);
    console.log(`Base fijada en ${uva.fecha} con UVA ${uva.valor}`);
  }

  const factor = Math.round((uva.valor / base.uva_base) * 10000) / 10000;
  const variacion = Math.abs(factor / previo.factor - 1);

  await guardar(FACTOR, {
    factor,
    etiqueta: etiqueta(uva.fecha),
    uva: uva.valor,
    uva_base: base.uva_base,
    fecha_uva: uva.fecha,
    actualizado: new Date().toISOString().slice(0, 10)
  });

  const cambio = factor !== previo.factor;
  const salto = variacion > UMBRAL;

  console.log(`factor ${previo.factor} -> ${factor} (${(variacion * 100).toFixed(1)}%)`);
  if (salto) console.log('Supera el umbral: va por pull request.');

  if (process.env.GITHUB_OUTPUT) {
    await writeFile(process.env.GITHUB_OUTPUT,
      `cambio=${cambio}\nsalto=${salto}\nfactor=${factor}\n`, { flag: 'a' });
  }
}

main().catch(e => {
  console.error('No se actualizo el factor:', e.message);
  process.exit(1);
});
