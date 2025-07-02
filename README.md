import React, { useState } from 'react';
import { Star, Trophy, TrendingUp, Users, Mail, Phone, Calendar } from 'lucide-react';
import MetricCard from '../components/Charts/MetricCard';
import BarChart from '../components/Charts/BarChart';
import DataTable from '../components/Tables/DataTable';
import DateFilter from '../components/Common/DateFilter';
import { mockSalesTeam, mockLeads, mockContracts } from '../data/mockData';

const Team: React.FC = () => {
  const [showDateFilter, setShowDateFilter] = useState(false);
  const [startDate, setStartDate] = useState('2024-01-01');
  const [endDate, setEndDate] = useState(new Date().toISOString().split('T')[0]);
  const [selectedPreset, setSelectedPreset] = useState('this_month');

  // Filter data by date range
  const filterByDateRange = (data: any[], dateField: string) => {
    return data.filter(item => {
      const itemDate = new Date(item[dateField]);
      const start = new Date(startDate);
      const end = new Date(endDate);
      return itemDate >= start && itemDate <= end;
    });
  };

  const filteredLeads = filterByDateRange(mockLeads, 'createdAt');
  const filteredContracts = filterByDateRange(mockContracts, 'createdAt');

  // Calculate team performance metrics
  const teamPerformance = mockSalesTeam.map(person => {
    const personLeads = filteredLeads.filter(lead => lead.assignedTo === person.name);
    const personContracts = filteredContracts.filter(contract => contract.salesPerson === person.name);
    const conversionRate = personLeads.length > 0 ? (personContracts.length / personLeads.length) * 100 : 0;
    const totalRevenue = personContracts.reduce((sum, contract) => sum + contract.value, 0);

    return {
      ...person,
      totalLeads: personLeads.length,
      closedContracts: personContracts.length,
      conversionRate: conversionRate.toFixed(1),
      totalRevenue,
      avgDealSize: personContracts.length > 0 ? (totalRevenue / personContracts.length).toFixed(0) : '0'
    };
  });

  const teamColumns = [
    { 
      key: 'name', 
      label: 'Nome',
      render: (name: string, row: any) => (
        <div className="flex items-center space-x-3">
          <div className="h-10 w-10 bg-blue-600 rounded-full flex items-center justify-center">
            <span className="text-white font-medium text-sm">
              {name.split(' ').map(n => n[0]).join('')}
            </span>
          </div>
          <div>
            <p className="text-sm font-medium text-gray-900">{name}</p>
            <p className="text-xs text-gray-500">{row.email}</p>
          </div>
        </div>
      )
    },
    { 
      key: 'role', 
      label: 'Cargo',
      render: (role: string) => {
        const roleLabels = {
          vendedor: 'Vendedor',
          gestor: 'Gestor',
          admin: 'Administrador'
        };
        const roleColors = {
          vendedor: 'bg-blue-100 text-blue-800',
          gestor: 'bg-green-100 text-green-800',
          admin: 'bg-purple-100 text-purple-800'
        };
        return (
          <span className={`px-2 py-1 rounded-full text-xs font-medium ${roleColors[role as keyof typeof roleColors]}`}>
            {roleLabels[role as keyof typeof roleLabels]}
          </span>
        );
      }
    },
    { key: 'totalLeads', label: 'Leads', sortable: true },
    { key: 'closedContracts', label: 'Contratos', sortable: true },
    { 
      key: 'conversionRate', 
      label: 'Taxa Conversão',
      render: (rate: string) => `${rate}%`
    },
    { 
      key: 'totalRevenue', 
      label: 'Receita Total',
      render: (revenue: number) => `R$ ${revenue.toLocaleString('pt-BR')}`
    },
    { 
      key: 'avgDealSize', 
      label: 'Ticket Médio',
      render: (avg: string) => `R$ ${parseInt(avg).toLocaleString('pt-BR')}`
    }
  ];

  const contractsData = teamPerformance.map(person => ({
    label: person.name.split(' ')[0],
    value: person.closedContracts,
    color: 'bg-blue-500'
  }));

  const revenueData = teamPerformance.map(person => ({
    label: person.name.split(' ')[0],
    value: Math.round(person.totalRevenue / 1000), // Convert to thousands
    color: 'bg-green-500'
  }));

  const topPerformer = teamPerformance.reduce((top, current) => 
    current.closedContracts > top.closedContracts ? current : top
  );

  const totalTeamRevenue = teamPerformance.reduce((sum, person) => sum + person.totalRevenue, 0);
  const totalTeamContracts = teamPerformance.reduce((sum, person) => sum + person.closedContracts, 0);
  const totalTeamLeads = teamPerformance.reduce((sum, person) => sum + person.totalLeads, 0);
  const avgTeamConversion = totalTeamLeads > 0 ? ((totalTeamContracts / totalTeamLeads) * 100).toFixed(1) : '0';

  return (
    <div className="p-6 space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold text-gray-900">Desempenho da Equipe</h1>
        <div className="flex items-center space-x-3">
          <button
            onClick={() => setShowDateFilter(!showDateFilter)}
            className={`flex items-center space-x-2 px-4 py-2 rounded-lg border ${
              showDateFilter 
                ? 'bg-blue-600 text-white border-blue-600' 
                : 'bg-white text-gray-700 border-gray-300 hover:bg-gray-50'
            }`}
          >
            <Calendar size={16} />
            <span>Filtro por Data</span>
          </button>
          <div className="flex items-center space-x-2 px-3 py-1 bg-yellow-100 text-yellow-800 rounded-full">
            <Trophy size={16} />
            <span className="text-sm font-medium">Top: {topPerformer.name}</span>
          </div>
        </div>
      </div>

      {/* Date Filter */}
      {showDateFilter && (
        <DateFilter
          startDate={startDate}
          endDate={endDate}
          onStartDateChange={setStartDate}
          onEndDateChange={setEndDate}
          onPresetSelect={setSelectedPreset}
        />
      )}

      {/* Period Summary */}
      {showDateFilter && (
        <div className="bg-blue-50 border border-blue-200 rounded-lg p-4">
          <div className="flex items-center space-x-3">
            <Calendar className="text-blue-600" size={20} />
            <div>
              <h3 className="text-sm font-medium text-blue-800">
                📊 Performance do Período: {new Date(startDate).toLocaleDateString('pt-BR')} até {new Date(endDate).toLocaleDateString('pt-BR')}
              </h3>
              <p className="text-sm text-blue-700 mt-1">
                {totalTeamLeads} leads trabalhados e {totalTeamContracts} contratos fechados pela equipe
              </p>
            </div>
          </div>
        </div>
      )}

      {/* Métricas Gerais da Equipe */}
      <div className="grid grid-cols-1 md:grid-cols-4 gap-6">
        <MetricCard
          title="Total de Membros"
          value={mockSalesTeam.length}
          icon={Users}
          color="blue"
        />
        <MetricCard
          title="Contratos da Equipe"
          value={totalTeamContracts}
          icon={Trophy}
          color="green"
        />
        <MetricCard
          title="Receita Total"
          value={`R$ ${totalTeamRevenue.toLocaleString('pt-BR')}`}
          icon={TrendingUp}
          color="green"
        />
        <MetricCard
          title="Conversão Média"
          value={`${avgTeamConversion}%`}
          icon={Star}
          color="yellow"
        />
      </div>

      {/* Destaque do Top Performer */}
      <div className="bg-gradient-to-r from-blue-500 to-purple-600 rounded-lg p-6 text-white">
        <div className="flex items-center justify-between">
          <div>
            <h3 className="text-lg font-semibold mb-2">🏆 Destaque do Período</h3>
            <p className="text-2xl font-bold">{topPerformer.name}</p>
            <p className="text-blue-100">Melhor performance em contratos fechados</p>
          </div>
          <div className="text-right">
            <div className="bg-white bg-opacity-20 rounded-lg p-4">
              <p className="text-3xl font-bold">{topPerformer.closedContracts}</p>
              <p className="text-sm text-blue-100">Contratos Fechados</p>
            </div>
          </div>
        </div>
        <div className="mt-4 grid grid-cols-3 gap-4 text-center">
          <div>
            <p className="text-2xl font-bold">{topPerformer.totalLeads}</p>
            <p className="text-xs text-blue-100">Leads Trabalhados</p>
          </div>
          <div>
            <p className="text-2xl font-bold">{topPerformer.conversionRate}%</p>
            <p className="text-xs text-blue-100">Taxa Conversão</p>
          </div>
          <div>
            <p className="text-2xl font-bold">R$ {topPerformer.totalRevenue.toLocaleString('pt-BR')}</p>
            <p className="text-xs text-blue-100">Receita Gerada</p>
          </div>
        </div>
      </div>

      {/* Gráficos de Performance */}
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <BarChart
          title="Contratos Fechados por Vendedor"
          data={contractsData}
        />
        <BarChart
          title="Receita por Vendedor (em milhares)"
          data={revenueData}
        />
      </div>

      {/* Tabela Detalhada da Equipe */}
      <DataTable
        data={teamPerformance}
        columns={teamColumns}
        title="Desempenho Individual Detalhado"
      />

      {/* Cards de Contato da Equipe */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
        {mockSalesTeam.map((member) => (
          <div key={member.id} className="bg-white rounded-lg shadow-sm border border-gray-200 p-4">
            <div className="flex items-center space-x-3 mb-3">
              <div className="h-12 w-12 bg-blue-600 rounded-full flex items-center justify-center">
                <span className="text-white font-medium">
                  {member.name.split(' ').map(n => n[0]).join('')}
                </span>
              </div>
              <div>
                <h4 className="font-medium text-gray-900">{member.name}</h4>
                <p className="text-sm text-gray-500 capitalize">{member.role}</p>
              </div>
            </div>
            <div className="space-y-2">
              <div className="flex items-center space-x-2 text-sm text-gray-600">
                <Mail size={14} />
                <span>{member.email}</span>
              </div>
              <div className="flex items-center space-x-2 text-sm text-gray-600">
                <Phone size={14} />
                <span>(11) 9999-9999</span>
              </div>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
};

export default Team;
