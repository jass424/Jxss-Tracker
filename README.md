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




