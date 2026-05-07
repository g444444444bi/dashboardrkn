import React, { useState, useMemo } from 'react';

/**
 * Dashboard de Gestão de Pagamentos para Prestadores
 * Regras: 
 * - Repasse Prestador: R$ 800,00 por loja
 * - Recebimento BK: R$ 1.450,00 por loja
 * - Prazo Repasse: 25/05/2024
 */

// --- CONSTANTES GLOBAIS ---
const REPASSE_FIXO = 800; 
const RECEBIMENTO_FIXO = 1450; 
const DATA_LIMITE_PAGAMENTO = new Date(2024, 4, 25); // Janeiro é 0, logo Maio é 4
const DIAS_PAGAMENTO_BK = [3, 9, 17, 24, 30];

// --- ÍCONES SVG (Para evitar erros de biblioteca) ---
const IconUsers = () => (
  <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M16 21v-2a4 4 0 0 0-4-4H6a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M22 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></svg>
);
const IconCalendar = () => (
  <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><rect width="18" height="18" x="3" y="4" rx="2" ry="2"/><line x1="16" x2="16" y1="2" y2="6"/><line x1="8" x2="8" y1="2" y2="6"/><line x1="3" x2="21" y1="10" y2="10"/></svg>
);
const IconWallet = () => (
  <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"><path d="M19 7V4a1 1 0 0 0-1-1H5a2 2 0 0 0 0 4h15a1 1 0 0 1 1 1v4h-3a2 2 0 0 0 0 4h3a1 1 0 0 0 1-1v-2a1 1 0 0 0-1-1"/><path d="M3 5v14a2 2 0 0 0 2 2h15a1 1 0 0 0 1-1v-4"/></svg>
);

// --- FUNÇÕES AUXILIARES ---
const formatarMoeda = (valor) => {
  return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(valor || 0);
};

const calcularPrevisaoBK = (dataServicoStr) => {
  if (!dataServicoStr) return null;
  const data = new Date(dataServicoStr);
  if (isNaN(data.getTime())) return null;
  
  data.setDate(data.getDate() + 30);
  const dia = data.getDate();
  const mes = data.getMonth();
  const ano = data.getFullYear();

  let diaFixo = DIAS_PAGAMENTO_BK.find(d => d >= dia);
  let dataFinal;

  if (!diaFixo) {
    dataFinal = new Date(ano, mes + 1, 3);
  } else {
    dataFinal = new Date(ano, mes, diaFixo);
  }
  return dataFinal;
};

// --- COMPONENTES ---
const CartaoPrestador = ({ prestador, isExpandido, onToggle }) => {
  const resumo = useMemo(() => {
    return prestador.lojas.reduce((acc, loja) => {
      const dataBK = calcularPrevisaoBK(loja.dataServico);
      const isAte25 = dataBK && dataBK <= DATA_LIMITE_PAGAMENTO;
      
      acc.totalPagar += loja.valorPagar;
      acc.totalReceber += loja.valorReceber;
      if (isAte25) {
        acc.qtdAte25++;
      } else {
        acc.qtdDps25++;
      }
      return acc;
    }, { totalPagar: 0, totalReceber: 0, qtdAte25: 0, qtdDps25: 0 });
  }, [prestador]);

  return (
    <div className="bg-white rounded-3xl shadow-sm border border-slate-100 overflow-hidden mb-4 transition-all hover:border-slate-300">
      <div className="p-6 cursor-pointer flex flex-col md:flex-row md:items-center justify-between gap-4" onClick={onToggle}>
        <div className="flex items-center gap-4">
          <div className={`p-3 rounded-2xl ${resumo.qtdDps25 > 0 ? 'bg-orange-50 text-orange-600' : 'bg-emerald-50 text-emerald-600'}`}>
            <IconUsers />
          </div>
          <div>
            <h3 className="text-lg font-black uppercase italic text-slate-800">{prestador.nome}</h3>
            <p className="text-xs font-bold text-slate-400 uppercase">{prestador.lojas.length} Lojas Realizadas</p>
          </div>
        </div>
        <div className="grid grid-cols-2 md:flex md:items-center gap-8">
          <div className="text-right">
            <p className="text-[10px] font-black text-slate-400 uppercase">Total Repasse</p>
            <p className="text-lg font-black text-slate-800">{formatarMoeda(resumo.totalPagar)}</p>
          </div>
          <div className="text-right">
            <p className="text-[10px] font-black text-slate-400 uppercase">Lucro</p>
            <p className="text-lg font-black text-emerald-600">+{formatarMoeda(resumo.totalReceber - resumo.totalPagar)}</p>
          </div>
        </div>
      </div>

      {isExpandido && (
        <div className="px-6 pb-6 bg-slate-50/30 border-t border-slate-50">
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4 py-6">
            <div className="bg-white p-4 rounded-2xl border border-emerald-100 flex justify-between items-center shadow-sm">
              <div>
                <p className="text-[10px] font-black text-emerald-600 uppercase">Recebe BK até 25/05</p>
                <p className="text-xl font-black text-slate-800">{resumo.qtdAte25} Lojas</p>
              </div>
            </div>
            <div className="bg-white p-4 rounded-2xl border border-orange-100 flex justify-between items-center shadow-sm">
              <div>
                <p className="text-[10px] font-black text-orange-600 uppercase">Recebe BK após 25/05</p>
                <p className="text-xl font-black text-slate-800">{resumo.qtdDps25} Lojas</p>
              </div>
            </div>
          </div>
          <div className="overflow-x-auto rounded-2xl border border-slate-200 bg-white">
            <table className="w-full text-left">
              <thead>
                <tr className="bg-slate-50 text-[10px] font-black text-slate-400 uppercase tracking-widest">
                  <th className="px-4 py-3">BKN</th>
                  <th className="px-4 py-3">Data Serviço</th>
                  <th className="px-4 py-3">Previsão BK</th>
                  <th className="px-4 py-3 text-right">A Pagar</th>
                  <th className="px-4 py-3 text-center">Status</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-slate-50">
                {prestador.lojas.map((loja, idx) => {
                  const dataBK = calcularPrevisaoBK(loja.dataServico);
                  const isAtrasado = dataBK && dataBK > DATA_LIMITE_PAGAMENTO;
                  return (
                    <tr key={idx} className="hover:bg-slate-50 transition-colors">
                      <td className="px-4 py-3 font-black text-sm text-slate-700">{loja.bkn}</td>
                      <td className="px-4 py-3 text-xs text-slate-500 font-bold">{new Date(loja.dataServico).toLocaleDateString('pt-BR')}</td>
                      <td className={`px-4 py-3 text-sm font-black ${isAtrasado ? 'text-orange-600' : 'text-emerald-600'}`}>{dataBK?.toLocaleDateString('pt-BR')}</td>
                      <td className="px-4 py-3 text-right font-black text-slate-800">{formatarMoeda(loja.valorPagar)}</td>
                      <td className="px-4 py-3 text-center">
                        <span className={`px-3 py-1 rounded-full text-[9px] font-black uppercase ${isAtrasado ? 'bg-orange-100 text-orange-600' : 'bg-emerald-100 text-emerald-600'}`}>
                          {isAtrasado ? '30/05' : 'No Prazo'}
                        </span>
                      </td>
                    </tr>
                  );
                })}
              </tbody>
            </table>
          </div>
        </div>
      )}
    </div>
  );
};

export default function App() {
  const [expandido, setExpandido] = useState("JEFERSON");

  const DADOS_CONSOLIDADO = useMemo(() => [
    {
      nome: "JEFERSON",
      lojas: [
        ...Array.from({ length: 17 }).map((_, i) => ({
          bkn: `BK${(i + 1).toString().padStart(4, '0')}`,
          dataServico: "2024-04-05", 
          valorPagar: REPASSE_FIXO,
          valorReceber: RECEBIMENTO_FIXO
        })),
        { bkn: "BK4832", dataServico: "2024-04-26", valorPagar: REPASSE_FIXO, valorReceber: RECEBIMENTO_FIXO },
        { bkn: "BK3637", dataServico: "2024-04-27", valorPagar: REPASSE_FIXO, valorReceber: RECEBIMENTO_FIXO },
        { bkn: "BK0048", dataServico: "2024-04-28", valorPagar: REPASSE_FIXO, valorReceber: RECEBIMENTO_FIXO }
      ]
    }
  ], []);

  const totais = useMemo(() => {
    let pagar = 0, receber = 0, gap = 0;
    DADOS_CONSOLIDADO.forEach(p => {
      p.lojas.forEach(l => {
        pagar += l.valorPagar;
        receber += l.valorReceber;
        const prevBK = calcularPrevisaoBK(l.dataServico);
        if (prevBK && prevBK > DATA_LIMITE_PAGAMENTO) gap += l.valorPagar;
      });
    });
    return { pagar, receber, gap };
  }, [DADOS_CONSOLIDADO]);

  return (
    <div className="min-h-screen bg-[#fcfaf7] p-8 font-sans text-slate-900">
      <div className="max-w-6xl mx-auto">
        <div className="mb-10 flex justify-between items-end">
          <div>
            <div className="flex items-center gap-3 mb-2">
              <div className="bg-[#D62300] p-2 rounded-xl"><IconWallet /></div>
              <h1 className="text-3xl font-black uppercase italic text-[#D62300]">Pagamentos BK</h1>
            </div>
            <p className="text-slate-500 font-bold flex items-center gap-2"><IconCalendar /> Repasse: 25/05/2024</p>
          </div>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mb-10">
          <div className="bg-white p-6 rounded-3xl shadow-sm border border-slate-100">
            <p className="text-[10px] font-black text-slate-400 uppercase mb-1">Total Repasse</p>
            <h3 className="text-3xl font-black">{formatarMoeda(totais.pagar)}</h3>
          </div>
          <div className="bg-white p-6 rounded-3xl shadow-sm border border-slate-100">
            <p className="text-[10px] font-black text-slate-400 uppercase mb-1">Margem Lucro</p>
            <h3 className="text-3xl font-black text-emerald-600">{formatarMoeda(totais.receber - totais.pagar)}</h3>
          </div>
          <div className="bg-slate-900 p-6 rounded-3xl shadow-2xl text-white">
            <p className="text-[10px] font-black text-slate-400 uppercase mb-1">Gap Financeiro (25/05)</p>
            <h3 className="text-3xl font-black text-red-500">{formatarMoeda(totais.gap)}</h3>
          </div>
        </div>

        {DADOS_CONSOLIDADO.map(p => (
          <CartaoPrestador key={p.nome} prestador={p} isExpandido={expandido === p.nome} onToggle={() => setExpandido(expandido === p.nome ? null : p.nome)} />
        ))}
      </div>
    </div>
  );
}
