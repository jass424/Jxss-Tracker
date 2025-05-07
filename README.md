import React, { useState, useEffect } from 'react';
import { Download, ChevronDown, Home, Trash2, Plus,  ArrowDownRight, ArrowUpRight, Car, Map, Bike } from 'lucide-react';
import { jsPDF } from 'jspdf';
import autoTable from 'jspdf-autotable';
import './bank-dashboard.css';



const formatCurrency = (amount) => {
  return new Intl.NumberFormat('en-IN', {
    style: 'currency',
    currency: 'INR',
    maximumFractionDigits: 0
  }).format(amount);
};

const formatDate = (date) => {
  return new Date(date).toLocaleDateString('en-IN');
};

// Transaction Form Component
function TransactionForm({ show, formData, onClose, onChange, onSubmit }) {
  if (!show) return null;

  const handleBackdropClick = (e) => {
    if (e.target === e.currentTarget) {
      onClose();
    }
  };

  return (
    <div className="modal-backdrop" onClick={handleBackdropClick}>
      <div className="bg-white rounded-lg shadow-xl max-w-md w-full mx-4">
        <div className="p-6 border-b border-gray-200">
          <h3 className="text-lg font-medium text-gray-800">Add New Transaction</h3>
        </div>
        
        <form className="p-6" onSubmit={onSubmit}>
          <div className="space-y-4">
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-1">Transaction Type</label>
              <div className="flex gap-4">
                <label className="inline-flex items-center">
                  <input 
                    type="radio" 
                    name="type" 
                    value="income" 
                    checked={formData.type === 'income'}
                    onChange={onChange}
                    className="h-4 w-4 text-blue-600 focus:ring-blue-500 border-gray-300 rounded" 
                  />
                  <span className="ml-2">Income</span>
                </label>
                <label className="inline-flex items-center">
                  <input 
                    type="radio" 
                    name="type" 
                    value="expense" 
                    checked={formData.type === 'expense'}
                    onChange={onChange}
                    className="h-4 w-4 text-blue-600 focus:ring-blue-500 border-gray-300 rounded" 
                  />
                  <span className="ml-2">Expense</span>
                </label>
              </div>
            </div>
            
            <div>
              <label htmlFor="amount" className="block text-sm font-medium text-gray-700 mb-1">Amount</label>
              <div className="relative rounded-md shadow-sm">
                <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                  <span className="text-gray-500 sm:text-sm">₹</span>
                </div>
                <input
                  type="number"
                  name="amount"
                  id="amount"
                  value={formData.amount}
                  onChange={onChange}
                  className="focus:ring-blue-500 focus:border-blue-500 block w-full pl-7 pr-12 sm:text-sm border-gray-300 rounded-md py-2 border"
                  placeholder="0.00"
                  required
                />
              </div>
            </div>
            
            <div>
              <label htmlFor="category" className="block text-sm font-medium text-gray-700 mb-1">Category</label>
              <select
                id="category"
                name="category"
                value={formData.category}
                onChange={onChange}
                className="mt-1 block w-full rounded-md border border-gray-300 py-2 px-3 shadow-sm focus:border-blue-500 focus:outline-none focus:ring-blue-500 sm:text-sm"
              >
                <option value="salary">Salary</option>
                <option value="freelance">Freelance</option>
                <option value="loans">Loans</option>
                <option value="utilities">Utilities</option>
                <option value="groceries">Groceries</option>
                <option value="dining">Dining</option>
                <option value="entertainment">Entertainment</option>
                <option value="travel">Travel</option>
                <option value="other">Other</option>
              </select>
            </div>
            
            <div>
              <label htmlFor="date" className="block text-sm font-medium text-gray-700 mb-1">Date</label>
              <input
                type="date"
                name="date"
                id="date"
                value={formData.date}
                onChange={onChange}
                className="mt-1 block w-full rounded-md border border-gray-300 py-2 px-3 shadow-sm focus:border-blue-500 focus:outline-none focus:ring-blue-500 sm:text-sm"
                required
              />
            </div>
            
            <div>
              <label htmlFor="description" className="block text-sm font-medium text-gray-700 mb-1">Description</label>
              <input
                type="text"
                name="description"
                id="description"
                value={formData.description}
                onChange={onChange}
                className="mt-1 block w-full rounded-md border border-gray-300 py-2 px-3 shadow-sm focus:border-blue-500 focus:outline-none focus:ring-blue-500 sm:text-sm"
                placeholder="Transaction description"
                required
              />
            </div>
          </div>
          
          <div className="mt-6 flex justify-end gap-3">
            <button
              type="button"
              onClick={onClose}
              className="px-4 py-2 bg-gray-100 text-gray-800 rounded-md hover:bg-gray-200 transition-colors"
            >
              Cancel
            </button>
            <button
              type="submit"
              className="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 transition-colors"
            >
              Add Transaction
            </button>
          </div>
        </form>
      </div>
    </div>
  );
}

// Transaction List Component
function TransactionList({
  transactions,
  onAddTransaction,
  onDeleteTransaction,
  showFilterOptions,
  toggleFilterOptions,
  filterMonth,
  filterCategory,
  onFilterChange
}) {
  const filterDropdownRef = React.useRef(null);

  // Close filter dropdown when clicking outside
  useEffect(() => {
    function handleClickOutside(event) {
      if (filterDropdownRef.current && 
          !filterDropdownRef.current.contains(event.target) && 
          showFilterOptions) {
        toggleFilterOptions();
      }
    }

    document.addEventListener("mousedown", handleClickOutside);
    return () => {
      document.removeEventListener("mousedown", handleClickOutside);
    };
  }, [showFilterOptions, toggleFilterOptions]);

  return (
    <div className="space-y-6">
      {/* Actions & Filters */}
      <div className="bg-white rounded-lg shadow p-6">
        <div className="flex flex-wrap justify-between items-center gap-4">
          <h2 className="text-lg font-medium text-gray-800">All Transactions</h2>
          
          <div className="flex flex-wrap gap-3">
            <button 
              className="bg-blue-600 hover:bg-blue-700 text-white rounded-md px-4 py-2 flex items-center gap-2 transition-colors"
              onClick={onAddTransaction}
            >
              <Plus size={18} />
              Add Transaction
            </button>
            
            <div className="relative">
              <button 
                className="bg-gray-200 hover:bg-gray-300 text-gray-800 rounded-md px-4 py-2 flex items-center gap-2 transition-colors"
                onClick={toggleFilterOptions}
              >
                Filter
                <ChevronDown size={18} />
              </button>
              
              {/* Filter dropdown */}
              {showFilterOptions && (
                <div ref={filterDropdownRef} className="absolute right-0 mt-2 w-56 bg-white rounded-md shadow-lg z-10">
                  <div className="p-3 border-b border-gray-100">
                    <label className="block text-sm font-medium text-gray-700 mb-1">Month</label>
                    <select 
                      className="block w-full rounded-md border-gray-300 shadow-sm py-2 px-3 bg-white text-gray-700 border"
                      value={filterMonth}
                      onChange={(e) => onFilterChange('filterMonth', e.target.value)}
                    >
                      <option value="all">All Months</option>
                      <option value="current">Current Month</option>
                      <option value="last">Last Month</option>
                    </select>
                  </div>
                  
                  <div className="p-3">
                    <label className="block text-sm font-medium text-gray-700 mb-1">Category</label>
                    <select 
                      className="block w-full rounded-md border-gray-300 shadow-sm py-2 px-3 bg-white text-gray-700 border"
                      value={filterCategory}
                      onChange={(e) => onFilterChange('filterCategory', e.target.value)}
                    >
                      <option value="all">All Categories</option>
                      <option value="salary">Salary</option>
                      <option value="freelance">Freelance</option>
                      <option value="loans">Loans</option>
                      <option value="utilities">Utilities</option>
                      <option value="groceries">Groceries</option>
                      <option value="dining">Dining</option>
                      <option value="entertainment">Entertainment</option>
                      <option value="travel">Travel</option>
                      <option value="other">Other</option>
                    </select>
                  </div>
                </div>
              )}
            </div>
          </div>
        </div>
      </div>
      
      {/* Transactions List */}
      <div className="bg-white rounded-lg shadow">
        <div className="overflow-x-auto">
          <table className="min-w-full divide-y divide-gray-200">
            <thead className="bg-gray-50">
              <tr>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Type</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Description</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Category</th>
                <th scope="col" className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Date</th>
                <th scope="col" className="px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">Amount</th>
                <th scope="col" className="px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">Actions</th>
              </tr>
            </thead>
            <tbody className="bg-white divide-y divide-gray-200">
              {transactions.length > 0 ? (
                transactions.map(transaction => (
                  <tr key={transaction.id}>
                    <td className="px-6 py-4 whitespace-nowrap">
                      <div className="flex items-center">
                        <div className={`flex-shrink-0 h-8 w-8 flex items-center justify-center rounded-full ${
                          transaction.type === 'income' ? 'bg-green-100 text-green-600' : 'bg-red-100 text-red-600'
                        }`}>
                          {transaction.type === 'income' ? 
                            <ArrowDownRight size={14} /> : 
                            <ArrowUpRight size={14} />
                          }
                        </div>
                        <div className="ml-3">
                          <div className="text-sm font-medium text-gray-900">
                            {transaction.type === 'income' ? 'Income' : 'Expense'}
                          </div>
                        </div>
                      </div>
                    </td>
                    <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-500">{transaction.description}</td>
                    <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-500">{transaction.category}</td>
                    <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-500">{formatDate(transaction.date)}</td>
                    <td className={`px-6 py-4 whitespace-nowrap text-sm font-medium text-right ${
                      transaction.type === 'income' ? 'text-green-600' : 'text-red-600'
                    }`}>
                      {transaction.type === 'income' ? 
                        formatCurrency(transaction.amount) : 
                        `-${formatCurrency(transaction.amount)}`
                      }
                    </td>
                    <td className="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
                      <button 
                        className="text-gray-400 hover:text-red-500 transition-colors"
                        onClick={() => onDeleteTransaction(transaction.id)}
                      >
                        <Trash2 size={18} />
                      </button>
                    </td>
                  </tr>
                ))
              ) : (
                <tr>
                  <td colSpan={6} className="px-6 py-10 text-center text-gray-500">
                    No transactions found. Add a transaction to get started.
                  </td>
                </tr>
              )}
            </tbody>
          </table>
        </div>
      </div>
    </div>
  );
}

// Loans List Component
function LoansList({ loans, totalLoanAmount, remainingLoanAmount, monthlyInstallment }) {
  // Get the appropriate icon for a loan type
  const getLoanIcon = (loanName) => {
    if (loanName.toLowerCase().includes('home')) return <Home size={16} className="mr-2" />;
    if (loanName.toLowerCase().includes('car')) return <Car size={16} className="mr-2" />;
    if (loanName.toLowerCase().includes('plot')) return <Map size={16} className="mr-2" />;
    if (loanName.toLowerCase().includes('bike')) return <Bike size={16} className="mr-2" />;
    return <Map size={16} className="mr-2" />; // Default icon
  };

  return (
    <div className="space-y-6">
      {/* Loan Summary */}
      <div className="bg-white rounded-lg shadow p-6">
        <h2 className="text-lg font-medium text-gray-800 mb-4">Loan Summary</h2>
        <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
          <div className="p-4 bg-green-50 rounded-lg">
            <p className="text-gray-500 ">Total Loan Amount</p>
            <p className="text-2xl font-bold text-gray-800">{formatCurrency(totalLoanAmount)}</p>
          </div>
          <div className="p-4 bg-green-50 rounded-lg">
            <p className="text-gray-500">Remaining Amount</p>
            <p className="text-2xl font-bold text-gray-800">{formatCurrency(remainingLoanAmount)}</p>
          </div>
          <div className="p-4 bg-green-50 rounded-lg">
            <p className="text-gray-500">Monthly Installment</p>
            <p className="text-2xl font-bold text-gray-800">{formatCurrency(monthlyInstallment)}</p>
          </div>
        </div>
      </div>
      
      {/* Active Loans */}
      <div className="bg-white rounded-lg shadow">
        <div className="px-6 py-4 border-b border-gray-200">
          <h2 className="text-x-lg font-medium text-gray-800 ">Active Loans</h2>
        </div>
        
        {loans.map(loan => {
          const progressPercentage = (loan.paidInstallments / loan.totalInstallments) * 100;
          
          return (
            <div key={loan.id} className="p-6 border-b border-gray-100">
              <div className="flex flex-wrap justify-between gap-4">
                <div>
                  <h3 className="text-lg font-medium text-gray-800">{loan.name}</h3>
                  <p className="text-gray-500 mt-1">
                    <span className="inline-flex items-center">
                      {getLoanIcon(loan.name)}
                      Installment: {formatCurrency(loan.installment)}/month
                    </span>
                  </p>
                </div>
                <div className="text-right">
                  <p className="text-lg font-medium text-gray-800">{formatCurrency(loan.remaining)}</p>
                  <p className="text-sm text-gray-500">remaining of {formatCurrency(loan.totalAmount)}</p>
                </div>
              </div>
              
              <div className="mt-4">
                <div className="flex justify-between text-sm mb-1">
                  <span>Progress: {loan.paidInstallments} of {loan.totalInstallments} installments paid</span>
                  <span>{progressPercentage.toFixed(2)}% complete</span>
                </div>
                <div className="w-full bg-gray-200 rounded-full h-2.5">
                  <div 
                    className="bg-blue-600 h-2.5 rounded-full" 
                    style={{ width: `${progressPercentage}%` }}
                  ></div>
                </div>
              </div>
            </div>
          );
        })}
      </div>
    </div>
  );
}

// Dashboard Overview Component
function DashboardOverview({
  balance,
  totalIncome,
  totalExpense,
  transactions,
  loans,
  onAddTransaction,
  onExportCSV,
  onExportPDF,
  onDeleteTransaction,
  onViewAllTransactions,
  totalLoanAmount,
  remainingLoanAmount,
  monthlyInstallment
}) {
  // Calculate spending percentages
  const loanPercentage = (monthlyInstallment / totalExpense) * 100;
  
  // Group expenses by category
  const expensesByCategory = transactions
    .filter(t => t.type === 'expense')
    .reduce((acc, curr) => {
      if (!acc[curr.category]) {
        acc[curr.category] = 0;
      }
      acc[curr.category] += curr.amount;
      return acc;
    }, {});
  
  // Get recent transactions
  const recentTransactions = transactions.slice(0, 5);

  return (
    <div className="space-y-6">
      {/* Balance card */}
      <div className="bg-white rounded-lg shadow p-6">
        <h2 className="c1">Current Balance</h2>
        <p className="c2">{formatCurrency(balance)}</p>
        <div className="mt-4 flex flex-wrap gap-6">
          <div>
            <p className="i1">Total Income</p>
            <p className="i2">{formatCurrency(totalIncome)}</p>
          </div>
          <div> 
            <p className="e1">Total Expenses</p>
            <p className="e2">{formatCurrency(totalExpense)}</p>
          </div>
        </div>
      </div>

      {/* Quick actions */}
      <div className="bg-white rounded-lg shadow p-6">
        <h2 className="q1">Quick Actions</h2>
        <div className="flex flex-wrap gap-3">
          <button 
            className="h1"
            onClick={onAddTransaction}
          >
            <Plus size={18} />
            Add Transaction
          </button>
          <button className="xl"
            onClick={onExportCSV}
          >
            <Download size={18} />
            Export CSV
          </button>
          <button className="p1"
            onClick={onExportPDF}
          >
            <Download size={18} />
            Export PDF
          </button>
        </div>
      </div>

      {/* Recent Transactions */}
      <div className="bg-white rounded-lg shadow p-6">
        <div className="flex justify-between items-center mb-4">
          <h2 className="text-lg font-medium text-gray-800">Recent Transactions</h2>
          <button 
            className="text-blue-600 text-sm font-medium hover:text-blue-800 transition-colors"
            onClick={onViewAllTransactions}
          >
            View All
          </button>
        </div>
        
        <div className="transaction-list">
          {recentTransactions.map(transaction => (
            <div key={transaction.id} className="py-3 border-b border-gray-100 flex justify-between items-center">
              <div className="flex items-start gap-3">
                <div className={`w-10 h-10 flex items-center justify-center rounded-full ${
                  transaction.type === 'income' ? 'bg-green-100 text-green-600' : 'bg-red-100 text-red-600'
                }`}>
                  {transaction.type === 'income' ? 
                    <ArrowDownRight size={18} /> : 
                    <ArrowUpRight size={18} />
                  }
                </div>
                <div>
                  <p className="font-medium text-gray-800">{transaction.description}</p>
                  <p className="text-sm text-gray-500">{transaction.category}</p>
                </div>
              </div>
              <div className="text-right">
                <p className={`font-medium ${
                  transaction.type === 'income' ? 'text-green-600' : 'text-red-600'
                }`}>
                  {transaction.type === 'income' ? 
                    formatCurrency(transaction.amount) : 
                    `-${formatCurrency(transaction.amount)}`
                  }
                </p>
                <p className="text-sm text-gray-500">{formatDate(transaction.date)}</p>
              </div>
              
            </div>
          ))}
          
          {recentTransactions.length === 0 && (
            <div className="py-8 text-center text-gray-500">
              No transactions yet. Add your first transaction to get started.
            </div>
          )}
        </div>
      </div>

      {/* Financial Summary */}
      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        {/* Loan Overview */}
        <div className="bg-white rounded-lg shadow p-6">
          <h2 className="text-xl font-medium text-Green-600 mb-4">Loan Overview</h2>
          <div className="space-y-4">
            <div>
              <p className="text-Black-500 text-xl font-medium">Total Loan Amount</p>
              <p className="text-xl font-medium text-red-600">{formatCurrency(totalLoanAmount)}</p>
            </div>
            <div>
              <p className="text-Black-500 text-xl font-medium">Remaining Amount</p>
              <p className="text-xl font-medium text-blue-600">{formatCurrency(remainingLoanAmount)}</p>
            </div>
            <div>
              <p className="text-Black-500 text-xl font-medium">Monthly Installment</p>
              <p className="text-xl font-medium text-green-600">{formatCurrency(monthlyInstallment)}</p>
            </div>
          </div>
        </div>

        {/* Monthly Spending */}
        <div className="bg-white rounded-lg shadow p-6">
          <h2 className="text-lg font-medium text-black-800 mb-4">Monthly Spending</h2>
          <div className="space-y-3">
            {Object.entries(expensesByCategory).map(([category, amount], index) => {
              const percentage = (amount / totalExpense) * 100;
              const colors = ['bg-blue-600', 'bg-green-500', 'bg-yellow-500', 'bg-red-500', 'bg-purple-500', 'bg-indigo-500'];
              const colorClass = colors[index % colors.length];
              
              return (
                <div key={category}>
                  <div className="flex justify-between">
                    <span className="text-gray-600">{category.charAt(0).toUpperCase() + category.slice(1)}</span>
                    <span className="font-medium">{formatCurrency(amount)}</span>
                  </div>
                  <div className="w-full bg-gray-200 rounded-full h-2.5">
                    <div className={`${colorClass} h-2.5 rounded-full`} style={{ width: `${percentage}%` }}></div>
                  </div>
                </div>
              );
            })}
            
            {Object.keys(expensesByCategory).length === 0 && (
              <div className="py-4 text-center text-gray-500">
                No expense data available yet.
              </div>
            )}
          </div>
        </div>
      </div>
    </div>
  );
}

export default function BankAccountDashboard() {
  // State management
  const [activeTab, setActiveTab] = useState('dashboard');
  const [transactions, setTransactions] = useState([]);
  const [balance, setBalance] = useState(0);
  const [loans, setLoans] = useState([]);
  const [showAddTransactionForm, setShowAddTransactionForm] = useState(false);
  const [showFilterOptions, setShowFilterOptions] = useState(false);
  const [filterMonth, setFilterMonth] = useState('all');
  const [filterCategory, setFilterCategory] = useState('all');
  const [transactionForm, setTransactionForm] = useState({
    type: 'income',
    amount: '',
    category: 'salary',
    date: new Date().toISOString().split('T')[0],
    description: ''
  });

  // Initialize demo data
  useEffect(() => {
    // Initial balance
    setBalance(150000);

    // Initial transactions
    const demoTransactions = [
      { id: 1, type: 'income', amount: 75000, category: 'salary', date: '2025-04-25', description: 'Monthly salary' },
      { id: 2, type: 'income', amount: 15000, category: 'freelance', date: '2025-04-10', description: 'Website development' },
      { id: 3, type: 'expense', amount: 20000, category: 'loans', date: '2025-04-05', description: 'Loan installments' },
      { id: 4, type: 'expense', amount: 8500, category: 'utilities', date: '2025-04-15', description: 'Electricity and water bills' },
      { id: 5, type: 'expense', amount: 12000, category: 'groceries', date: '2025-04-18', description: 'Monthly groceries' },
      { id: 6, type: 'expense', amount: 5000, category: 'dining', date: '2025-04-22', description: 'Dinner with family' },
    ];
    setTransactions(demoTransactions);

    // Initial loans
    const demoLoans = [
      { id: 1, name: 'Home Loan', totalAmount: 1000000, installment: 8000, remaining: 992000, paidInstallments: 3, totalInstallments: 120 },
      { id: 2, name: 'Car Loan', totalAmount: 500000, installment: 5000, remaining: 485000, paidInstallments: 3, totalInstallments: 60 },
      { id: 3, name: 'Plot Loan', totalAmount: 300000, installment: 4000, remaining: 288000, paidInstallments: 3, totalInstallments: 48 },
      { id: 4, name: 'Bike Loan', totalAmount: 200000, installment: 3000, remaining: 191000, paidInstallments: 3, totalInstallments: 36 },
    ];
    setLoans(demoLoans);
  }, []);

  // Handle form change
  const handleFormChange = (e) => {
    const { name, value } = e.target;
    setTransactionForm({
      ...transactionForm,
      [name]: value
    });
  };

  // Handle filter change
  const handleFilterChange = (filterType, value) => {
    if (filterType === 'filterMonth') {
      setFilterMonth(value);
    } else if (filterType === 'filterCategory') {
      setFilterCategory(value);
    }
  };

  // Handle add transaction
  const handleAddTransaction = () => {
    setShowAddTransactionForm(true);
  };

  // Handle submit transaction
  const handleSubmitTransaction = (e) => {
    e.preventDefault();
    
    const newTransaction = {
      id: Date.now(),
      type: transactionForm.type,
      amount: parseFloat(transactionForm.amount),
      category: transactionForm.category,
      date: transactionForm.date,
      description: transactionForm.description
    };
    
    setTransactions([newTransaction, ...transactions]);
    
    // Update balance
    if (newTransaction.type === 'income') {
      setBalance(prevBalance => prevBalance + newTransaction.amount);
    } else {
      setBalance(prevBalance => prevBalance - newTransaction.amount);
    }
    
    // Reset form and close modal
    setTransactionForm({
      type: 'income',
      amount: '',
      category: 'salary',
      date: new Date().toISOString().split('T')[0],
      description: ''
    });
    setShowAddTransactionForm(false);
  };

  // Handle delete transaction
  const handleDeleteTransaction = (id) => {
    const transactionToDelete = transactions.find(t => t.id === id);
    
    // Update balance
    if (transactionToDelete.type === 'income') {
      setBalance(prevBalance => prevBalance - transactionToDelete.amount);
    } else {
      setBalance(prevBalance => prevBalance + transactionToDelete.amount);
    }
    
    // Remove transaction
    setTransactions(transactions.filter(t => t.id !== id));
  };

  // Handle export to CSV
  const handleExportCSV = () => {
    const header = ['Type', 'Description', 'Category', 'Date', 'Amount'];
    
    const csvData = transactions.map(t => [
      t.type,
      t.description,
      t.category,
      formatDate(t.date),
      t.amount
    ]);
    
    const csvContent = [header, ...csvData]
      .map(row => row.join(','))
      .join('\n');
    
    const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.setAttribute('href', url);
    link.setAttribute('download', `transactions_${new Date().toISOString().slice(0, 10)}.csv`);
    link.click();
  };

  // Handle export to PDF - FIXED
  const handleExportPDF = () => {
    const doc = new jsPDF();

    doc.setFontSize(16);
    doc.text('Bank Account Transactions', 14, 15);

    doc.setFontSize(12);
    doc.text(`Balance: ${formatCurrency(balance)}`, 14, 25);
    doc.text(`Generated on: ${new Date().toLocaleDateString()}`, 14, 30);

    const tableColumn = ['Type', 'Description', 'Category', 'Date', 'Amount'];
    const tableRows = [];

    transactions.forEach(t => {
      const transactionData = [
        t.type === 'income' ? 'Income' : 'Expense',
        t.description,
        t.category,
        formatDate(t.date),
        t.type === 'income' ? 
          formatCurrency(t.amount) : 
          `-${formatCurrency(t.amount)}`
      ];
      tableRows.push(transactionData);
    });

    autoTable(doc, {
      head: [tableColumn],
      body: tableRows,
      startY: 35,
      styles: { fontSize: 8 },
      headStyles: { fillColor: [33, 150, 243] },
    });

    doc.save(`bank_transactions_${new Date().toISOString().slice(0, 10)}.pdf`);
  };

  // Calculate filtered transactions
  const filteredTransactions = transactions.filter(transaction => {
    const transactionDate = new Date(transaction.date);
    const currentDate = new Date();
    
    // Filter by month
    if (filterMonth === 'current') {
      if (
        transactionDate.getMonth() !== currentDate.getMonth() ||
        transactionDate.getFullYear() !== currentDate.getFullYear()
      ) {
        return false;
      }
    } else if (filterMonth === 'last') {
      const lastMonth = currentDate.getMonth() === 0 ? 11 : currentDate.getMonth() - 1;
      const lastMonthYear = currentDate.getMonth() === 0 ? 
        currentDate.getFullYear() - 1 : 
        currentDate.getFullYear();
      
      if (
        transactionDate.getMonth() !== lastMonth ||
        transactionDate.getFullYear() !== lastMonthYear
      ) {
        return false;
      }
    }
    
    // Filter by category
    if (filterCategory !== 'all' && transaction.category !== filterCategory) {
      return false;
    }
    
    return true;
  });

  // Calculate totals
  const totalIncome = transactions
    .filter(t => t.type === 'income')
    .reduce((sum, t) => sum + t.amount, 0);
    
  const totalExpense = transactions
    .filter(t => t.type === 'expense')
    .reduce((sum, t) => sum + t.amount, 0);
    
  const totalLoanAmount = loans.reduce((sum, loan) => sum + loan.totalAmount, 0);
  const remainingLoanAmount = loans.reduce((sum, loan) => sum + loan.remaining, 0);
  const monthlyInstallment = loans.reduce((sum, loan) => sum + loan.installment, 0);

  return (
    <div className="min-h-screen bg-gray-100 p-4 md:p-6">
      <div className="max-w-7xl mx-auto">
        {/* Header */}
        <div className="flex flex-col md:flex-row md:items-center justify-between mb-6">
          <div>
            <h1 className="j1">Jxss Account</h1>
          </div>
          
        </div>
        
        {/* Tabs */}
        <div className="bg-white rounded-lg shadow-sm mb-6">
          <div className="flex overflow-x-auto">
            <button className="d1"
              onClick={() => setActiveTab('dashboard')}
            >
              Dashboard
            </button>
            <button
              className="t1"
              onClick={() => setActiveTab('transactions')}
            >
              Transactions
            </button>
            <button
              className="l1"
              onClick={() => setActiveTab('loans')}
            >
              Loans
            </button>
          </div>
        </div>
        
        {/* Main Content */}
        <div>
          {activeTab === 'dashboard' && (
            <DashboardOverview
              balance={balance}
              totalIncome={totalIncome}
              totalExpense={totalExpense}
              transactions={transactions}
              loans={loans}
              onAddTransaction={handleAddTransaction}
              onExportCSV={handleExportCSV}
              onExportPDF={handleExportPDF}
              onDeleteTransaction={handleDeleteTransaction}
              onViewAllTransactions={() => setActiveTab('transactions')}
              totalLoanAmount={totalLoanAmount}
              remainingLoanAmount={remainingLoanAmount}
              monthlyInstallment={monthlyInstallment}
            />
          )}
          
          {activeTab === 'transactions' && (
            <TransactionList
              transactions={filteredTransactions}
              onAddTransaction={handleAddTransaction}
              onDeleteTransaction={handleDeleteTransaction}
              showFilterOptions={showFilterOptions}
              toggleFilterOptions={() => setShowFilterOptions(!showFilterOptions)}
              filterMonth={filterMonth}
              filterCategory={filterCategory}
              onFilterChange={handleFilterChange}
            />
          )}
          
          {activeTab === 'loans' && (
            <LoansList
              loans={loans}
              totalLoanAmount={totalLoanAmount}
              remainingLoanAmount={remainingLoanAmount}
              monthlyInstallment={monthlyInstallment}
            />
          )}
        </div>
      </div>
      
      {/* Transaction Form Modal */}
      <TransactionForm
        show={showAddTransactionForm}
        formData={transactionForm}
        onClose={() => setShowAddTransactionForm(false)}
        onChange={handleFormChange}
        onSubmit={handleSubmitTransaction}
      />
    </div>
  );
}




//Css File for this 

bank-dashboard.css

/* Reset some basics */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

body, html, #root {
  height: 100%;
  background-color: #f3f4f6; /* Tailwind gray-100 */
}

/* Centering the main container vertically and horizontally */
.min-h-screen {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: flex-start; /* top aligned, use center if vertical center needed */
  padding: 1rem;
  overflow-y: auto;
}

/* Max width and padding */
.max-w-7xl {
  width: 100%;
  max-width: 1200px;
}

/* Header */
.j1 {
  font-size: 2.5rem;
  font-weight: 700;
  color: #1e293b; /* Slate-800 */
  text-align: center;
  margin-bottom: 1.5rem;
}

/* Tabs container */
.bg-white.shadow-sm {
  border-radius: 0.5rem;
}

.bg-white.shadow-sm .flex {
  display: flex;
  justify-content: center;
  border-bottom: 2px solid #e5e7eb;
  overflow-x: auto;
  border-radius: 0.5rem;
}

/* Tab buttons */
.d1, .t1, .l1 {
  flex: 1 0 auto;
  padding: 0.75rem 1.5rem;
  font-weight: 600;
  background: none;
  border: none;
  cursor: pointer;
  color: #64748b; /* gray-500 */
  transition: all 0.3s ease;
  font-size: 1rem;
  border-bottom: 3px solid transparent;
  text-align: center;
  user-select: none;
  white-space: nowrap;
}

.d1:hover, .t1:hover, .l1:hover,
.d1:focus, .t1:focus, .l1:focus {
  color: #2563eb; /* blue-600 */
  outline: none;
  border-bottom: 3px solid #2563eb;
}

/* Active tab style */
.d1.active, .t1.active, .l1.active {
  color: #1d4ed8; /* blue-700 */
  border-bottom: 3px solid #1d4ed8;
  font-weight: 700;
}

/* Card styling */
.bg-white.rounded-lg.shadow {
  background-color: white;
  border-radius: 0.75rem;
  box-shadow: 0 4px 12px rgb(0 0 0 / 0.1);
  padding: 1.5rem;
  margin-bottom: 1.5rem;
}

/* Balance card */
.c1 {
  font-size: 1.3rem;
  font-weight: 700;
  color: #0f172a; /* Slate-900 */
  text-align: center;
  margin-bottom: 0.25rem;
}

.c2 {
  font-size: 2.5rem;
  font-weight: 700;
  color: #047857; /* Emerald-600 */
  text-align: center;
}

/* Income and Expenses */
.i1, .e1 {
  font-size: 1rem;
  font-weight: 600;
  margin-bottom: 0.25rem;
  color: #374151; /* Gray-700 */
}

.i2, .e2 {
  font-size: 1.5rem;
  font-weight: 700;
}

.i1, .i2 {
  color: #059669; /* Emerald-600 */
  text-align: center;
}

.e1, .e2 {
  color: #b91c1c; /* Red-700 */
  text-align: center;
}

.mt-4.flex.flex-wrap.gap-6 {
  display: flex;
  justify-content: center;
  gap: 3rem;
}

/* Quick action buttons */
.q1 {
  font-size: 1.25rem;
  font-weight: 700;
  color: #2563eb;
  margin-bottom: 1rem;
  text-align: center;
}

.flex.flex-wrap.gap-3 {
  display: flex;
  justify-content: center;
  gap: 1rem;
  flex-wrap: wrap;
}

/* Buttons */
.h1, .xl, .p1 {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 600;
  padding: 0.6rem 1.25rem;
  border-radius: 0.5rem;
  border: none;
  transition: background-color 0.3s;
  user-select: none;
  text-decoration: none;
}

/* Add Transaction */
.h1 {
  background-color: #2563eb;
  color: white;
}

.h1:hover {
  background-color: #1d4ed8;
}

/* Export CSV */
.xl {
  background-color: #1f2937; /* Gray-800 */
  color: white;
}

.xl:hover {
  background-color: #111827;
}

/* Export PDF */
.p1 {
  background-color: #047857;
  color: white;
}

.p1:hover {
  background-color: #065f46;
}

/* Transactions & Loans Table */
table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9rem;
}

th, td {
  padding: 0.75rem 1rem;
  border-bottom: 1px solid #e5e7eb; /* Gray-200 */
  text-align: left;
  color: #374151; /* Gray-700 */
  vertical-align: middle;
}

thead th {
  background-color: #f9fafb; /* Gray-50 */
  font-weight: 700;
  font-size: 0.85rem;
  color: #6b7280; /* Gray-500 */
  text-transform: uppercase;
}

tbody tr:hover {
  background-color: #f3f4f6; /* Gray-100 */
}

.text-right {
  text-align: right;
}

/* Transaction Type badges */
.flex-shrink-0.h-8.w-8 {
  width: 2rem;
  height: 2rem;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 9999px;
  font-weight: 700;
}

.bg-green-100 {
  background-color: #d1fae5;
  color: #047857;
}

.bg-red-100 {
  background-color: #fee2e2;
  color: #b91c1c;
}

/* Modal backdrop */
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(31, 41, 55, 0.75);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  padding: 1rem;
}

/* Modal box */
.modal-backdrop > div {
  max-width: 420px;
  width: 100%;
  background: white;
  border-radius: 0.75rem;
  box-shadow: 0 10px 15px rgb(0 0 0 / 0.1);
  overflow: hidden;
}

/* Modal header */
.modal-backdrop h3 {
  font-size: 1.25rem;
  font-weight: 600;
  color: #1f2937;
  margin-bottom: 0.5rem;
}

/* Form inputs */
.form-input,
select,
input[type="date"],
input[type="number"],
input[type="text"] {
  width: 100%;
  padding: 0.5rem 0.75rem;
  border: 1px solid #d1d5db; /* Gray-300 */
  border-radius: 0.5rem;
  font-size: 1rem;
  color: #374151; /* Gray-700 */
  transition: border-color 0.3s;
}

.form-input:focus,
select:focus,
input[type="date"]:focus,
input[type="number"]:focus,
input[type="text"]:focus {
  outline: none;
  border-color: #2563eb;
  box-shadow: 0 0 0 2px rgb(37 99 235 / 0.3);
}

/* Labels */
label {
  font-weight: 600;
  font-size: 0.875rem;
  color: #374151;
  margin-bottom: 0.25rem;
  display: block;
}

/* Form buttons */
.modal-backdrop button[type="submit"],
.modal-backdrop button[type="button"] {
  padding: 0.5rem 1.25rem;
  border-radius: 0.5rem;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.3s;
  border: none;
  display: inline-block;
}

.modal-backdrop button[type="button"] {
  background-color: #e5e7eb; /* Gray-200 */
  color: #374151;
  margin-right: 0.5rem;
}

.modal-backdrop button[type="button"]:hover {
  background-color: #d1d5db;
}

.modal-backdrop button[type="submit"] {
  background-color: #2563eb;
  color: white;
}

.modal-backdrop button[type="submit"]:hover {
  background-color: #1d4ed8;
}

.mt-6.flex.justify-end.gap-3 {
  display: flex;
  justify-content: flex-end;
  gap: 0.75rem;
  margin-top: 1.5rem;
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .flex-wrap {
    flex-wrap: wrap;
  }
  
  .px-6, .py-4 {
    padding-left: 1rem;
    padding-right: 1rem;
    padding-top: 0.75rem;
    padding-bottom: 0.75rem;
  }
  
  /* Smaller tab font for smaller screens */
  .d1, .t1, .l1 {
    font-size: 0.875rem;
    padding: 0.5rem 0.75rem;
  }
}


