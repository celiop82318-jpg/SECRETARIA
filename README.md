# SECRETARIA
import React, { useMemo, useState, useEffect } from "react";
import { motion } from "framer-motion";
import {
  LineChart,
  Line,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
  BarChart,
  Bar,
  Legend,
  PieChart,
  Pie,
  Cell,
} from "recharts";
import { Download, FileText, RefreshCcw, Filter, Printer } from "lucide-react";
import { Button } from "@/components/ui/button";
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import { Input } from "@/components/ui/input";

// ==========================
// UTILITÁRIOS
// ==========================
function formatNumber(n) {
  return new Intl.NumberFormat("pt-BR").format(n || 0);
}

function toCSV(rows) {
  if (!rows?.length) return "";
  const headers = Object.keys(rows[0]);
  const csv = [headers.join(",")]
    .concat(
      rows.map((r) => headers.map((h) => JSON.stringify(r[h] ?? "")).join(","))
    )
    .join("\n");
  return csv;
}

function downloadFile(filename, content, mime = "text/plain;charset=utf-8") {
  const blob = new Blob([content], { type: mime });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = filename;
  a.click();
  URL.revokeObjectURL(url);
}

// ==========================
// MOCK DE DADOS (substituir por API)
// ==========================
const unidades = [
  { id: "todas", nome: "Todas as Unidades" },
  { id: "ubs-centro", nome: "UBS Centro" },
  { id: "ubs-leste", nome: "UBS Leste" },
  { id: "hospital-municipal", nome: "Hospital Municipal" },
];

const baseMensal = [
  { mes: "Jan", consultas: 1320, vacinas: 820, internacoes: 44 },
  { mes: "Fev", consultas: 1275, vacinas: 910, internacoes: 39 },
  { mes: "Mar", consultas: 1502, vacinas: 980, internacoes: 51 },
  { mes: "Abr", consultas: 1410, vacinas: 1050, internacoes: 47 },
  { mes: "Mai", consultas: 1601, vacinas: 1120, internacoes: 49 },
  { mes: "Jun", consultas: 1550, vacinas: 1180, internacoes: 46 },
  { mes: "Jul", consultas: 1710, vacinas: 1210, internacoes: 55 },
  { mes: "Ago", consultas: 1665, vacinas: 1195, internacoes: 50 },
  { mes: "Set", consultas: 1730, vacinas: 1230, internacoes: 52 },
  { mes: "Out", consultas: 1680, vacinas: 1255, internacoes: 48 },
  { mes: "Nov", consultas: 1750, vacinas: 1270, internacoes: 54 },
  { mes: "Dez", consultas: 1625, vacinas: 1185, internacoes: 46 },
];

const mixAtendimentos = [
  { tipo: "Atenção Básica", valor: 58 },
  { tipo: "Urgência/Emergência", valor: 22 },
  { tipo: "Especialidades", valor: 14 },
  { tipo: "Internações", valor: 6 },
];

const pacientesTabela = Array.from({ length: 40 }).map((_, i) => ({
  protocolo: String(10000 + i),
  paciente: `Paciente ${i + 1}`,
  unidade: i % 3 === 0 ? "Hospital Municipal" : i % 2 === 0 ? "UBS Leste" : "UBS Centro",
  servico: i % 4 === 0 ? "Consulta" : i % 4 === 1 ? "Vacinação" : i % 4 === 2 ? "Exame" : "Atendimento Urgência",
  data: new Date(2025, (i % 12), (i % 27) + 1).toLocaleDateString("pt-BR"),
}));

// ==========================
// COMPONENTE PRINCIPAL
// ==========================
export default function DashboardSaude() {
  // Filtros de alto nível
  const [unidade, setUnidade] = useState("todas");
  const [dataDe, setDataDe] = useState("");
  const [dataAte, setDataAte] = useState("");

  // Estado para simular carregamento/refresh
  const [loading, setLoading] = useState(false);

  // Simulação de fetch — substituir por chamada real de API
  useEffect(() => {
    // no-op por enquanto; manter como placeholder de integração
  }, [unidade, dataDe, dataAte]);

  const kpis = useMemo(() => {
    // KPIs derivados do mock (exemplo)
    const totalConsultas = baseMensal.reduce((acc, m) => acc + m.consultas, 0);
    const totalVacinas = baseMensal.reduce((acc, m) => acc + m.vacinas, 0);
    const totalInternacoes = baseMensal.reduce((acc, m) => acc + m.internacoes, 0);
    const taxaConversaoVacinal = ((totalVacinas / (totalConsultas || 1)) * 100).toFixed(1);

    return [
      { titulo: "Consultas (12m)", valor: formatNumber(totalConsultas) },
      { titulo: "Vacinas (12m)", valor: formatNumber(totalVacinas) },
      { titulo: "Internações (12m)", valor: formatNumber(totalInternacoes) },
      { titulo: "% Vacinação/Consultas", valor: `${taxaConversaoVacinal}%` },
    ];
  }, []);

  const handleExportCSV = () => {
    const csv = toCSV(pacientesTabela);
    downloadFile(`relatorio-pacientes-${Date.now()}.csv`, csv, "text/csv;charset=utf-8");
  };

  const handlePrint = () => {
    window.print();
  };

  const handleRefresh = async () => {
    setLoading(true);
    // simula refetch
    await new Promise((r) => setTimeout(r, 800));
    setLoading(false);
  };

  return (
    <div className="min-h-screen w-full bg-gray-50 p-6">
      <header className="mb-6 flex flex-col gap-3 md:flex-row md:items-center md:justify-between">
        <div>
          <h1 className="text-2xl font-bold tracking-tight">Painel de Saúde • Gráficos & Relatórios</h1>
          <p className="text-sm text-gray-600">Monitore KPIs, tendências mensais e gere relatórios operacionais em um clique.</p>
        </div>
        <div className="flex items-center gap-2">
          <Button variant="outline" onClick={handleRefresh} disabled={loading}>
            <RefreshCcw className={`mr-2 h-4 w-4 ${loading ? "animate-spin" : ""}`} /> Atualizar
          </Button>
          <Button variant="outline" onClick={handlePrint}>
            <Printer className="mr-2 h-4 w-4" /> Imprimir/Salvar PDF
          </Button>
          <Button onClick={handleExportCSV}>
            <Download className="mr-2 h-4 w-4" /> Exportar CSV
          </Button>
        </div>
      </header>

      {/* Filtros */}
      <Card className="mb-6">
        <CardHeader className="pb-2">
          <CardTitle className="text-base">Filtros</CardTitle>
          <CardDescription>Refine a visão por unidade e período.</CardDescription>
        </CardHeader>
        <CardContent>
          <div className="grid grid-cols-1 gap-3 md:grid-cols-4">
            <div className="col-span-1">
              <label className="mb-1 block text-xs font-medium text-gray-700">Unidade</label>
              <Select value={unidade} onValueChange={setUnidade}>
                <SelectTrigger>
                  <SelectValue placeholder="Selecione a unidade" />
                </SelectTrigger>
                <SelectContent>
                  {unidades.map((u) => (
                    <SelectItem key={u.id} value={u.id}>{u.nome}</SelectItem>
                  ))}
                </SelectContent>
              </Select>
            </div>
            <div className="col-span-1">
              <label className="mb-1 block text-xs font-medium text-gray-700">Data Inicial</label>
              <Input type="date" value={dataDe} onChange={(e) => setDataDe(e.target.value)} />
            </div>
            <div className="col-span-1">
              <label className="mb-1 block text-xs font-medium text-gray-700">Data Final</label>
              <Input type="date" value={dataAte} onChange={(e) => setDataAte(e.target.value)} />
            </div>
            <div className="col-span-1 flex items-end">
              <Button className="w-full" variant="secondary">
                <Filter className="mr-2 h-4 w-4" /> Aplicar Filtros
              </Button>
            </div>
          </div>
        </CardContent>
      </Card>

      {/* KPIs */}
      <div className="mb-6 grid grid-cols-1 gap-4 md:grid-cols-4">
        {kpis.map((k, idx) => (
          <motion.div
            key={k.titulo}
            initial={{ opacity: 0, y: 8 }}
            animate={{ opacity: 1, y: 0 }}
            transition={{ delay: idx * 0.05 }}
          >
            <Card className="shadow-sm">
              <CardHeader className="pb-2">
                <CardDescription>{k.titulo}</CardDescription>
                <CardTitle className="text-2xl">{k.valor}</CardTitle>
              </CardHeader>
            </Card>
          </motion.div>
        ))}
      </div>

      {/* Gráficos */}
      <div className="grid grid-cols-1 gap-6 xl:grid-cols-3">
        <Card className="xl:col-span-2">
          <CardHeader className="pb-0">
            <CardTitle className="text-base">Consultas x Vacinas (Mensal)</CardTitle>
            <CardDescription>Volume mensal consolidado por 12 meses.</CardDescription>
          </CardHeader>
          <CardContent className="h-80 pt-2">
            <ResponsiveContainer width="100%" height="100%">
              <LineChart data={baseMensal} margin={{ top: 10, right: 20, left: 0, bottom: 0 }}>
                <CartesianGrid strokeDasharray="3 3" />
                <XAxis dataKey="mes" />
                <YAxis />
                <Tooltip />
                <Legend />
                <Line type="monotone" dataKey="consultas" strokeWidth={2} dot={false} />
                <Line type="monotone" dataKey="vacinas" strokeWidth={2} dot={false} />
              </LineChart>
            </ResponsiveContainer>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="pb-0">
            <CardTitle className="text-base">Mix de Atendimentos</CardTitle>
            <CardDescription>Distribuição percentual por tipo.</CardDescription>
          </CardHeader>
          <CardContent className="h-80 pt-2">
            <ResponsiveContainer width="100%" height="100%">
              <PieChart>
                <Pie data={mixAtendimentos} dataKey="valor" nameKey="tipo" outerRadius={100} label>
                  {mixAtendimentos.map((_, i) => (
                    <Cell key={`c-${i}`} />
                  ))}
                </Pie>
                <Tooltip />
                <Legend />
              </PieChart>
            </ResponsiveContainer>
          </CardContent>
        </Card>

        <Card className="xl:col-span-3">
          <CardHeader className="pb-0">
            <CardTitle className="text-base">Internações por Mês</CardTitle>
            <CardDescription>Monitoramento de ocupação e sazonalidade.</CardDescription>
          </CardHeader>
          <CardContent className="h-80 pt-2">
            <ResponsiveContainer width="100%" height="100%">
              <BarChart data={baseMensal} margin={{ top: 10, right: 20, left: 0, bottom: 0 }}>
                <CartesianGrid strokeDasharray="3 3" />
                <XAxis dataKey="mes" />
                <YAxis />
                <Tooltip />
                <Legend />
                <Bar dataKey="internacoes" />
              </BarChart>
            </ResponsiveContainer>
          </CardContent>
        </Card>
      </div>

      {/* Tabela */}
      <Card className="mt-6">
        <CardHeader className="pb-0">
          <div className="flex items-center justify-between">
            <div>
              <CardTitle className="text-base">Relatório Operacional — Atendimentos</CardTitle>
              <CardDescription>Lista de registros com protocolo, paciente, unidade e serviço.</CardDescription>
            </div>
            <div className="flex items-center gap-2">
              <Button size="sm" variant="outline" onClick={handleExportCSV}>
                <Download className="mr-2 h-4 w-4" /> CSV
              </Button>
              <Button size="sm" variant="outline" onClick={handlePrint}>
                <FileText className="mr-2 h-4 w-4" /> PDF (Imprimir)
              </Button>
            </div>
          </div>
        </CardHeader>
        <CardContent className="pt-4">
          <div className="overflow-auto rounded-xl border">
            <table className="min-w-full text-sm">
              <thead className="bg-gray-100">
                <tr>
                  <th className="px-3 py-2 text-left font-medium">Protocolo</th>
                  <th className="px-3 py-2 text-left font-medium">Paciente</th>
                  <th className="px-3 py-2 text-left font-medium">Unidade</th>
                  <th className="px-3 py-2 text-left font-medium">Serviço</th>
                  <th className="px-3 py-2 text-left font-medium">Data</th>
                </tr>
              </thead>
              <tbody>
                {pacientesTabela.map((row) => (
                  <tr key={row.protocolo} className="border-t">
                    <td className="px-3 py-2">{row.protocolo}</td>
                    <td className="px-3 py-2">{row.paciente}</td>
                    <td className="px-3 py-2">{row.unidade}</td>
                    <td className="px-3 py-2">{row.servico}</td>
                    <td className="px-3 py-2">{row.data}</td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </CardContent>
      </Card>

      {/* Rodapé */}
      <div className="mt-8 text-center text-xs text-gray-500">
        v1.0 • Arquitetura pronta para integrar com API (FastAPI/Node). Exporte CSV e imprima relatórios.
      </div>
    </div>
  );
}
