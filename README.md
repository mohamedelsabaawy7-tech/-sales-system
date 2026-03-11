import React, { useState, useEffect, useRef } from ‘react’;
import {
LayoutDashboard, ShoppingCart, Wallet, Target, FileText, Bell, Plus, Trash2,
CheckCircle2, TrendingUp, AlertCircle, Upload, Zap, BarChart3, ArrowUpRight,
ArrowDownRight, ChevronRight, X, RefreshCw, Package, DollarSign,
Activity, Search, Eye, PieChart, Layers, Clock, Printer,
Users, Shield, Lock, UserCheck, Truck, CreditCard, Receipt,
LogIn, LogOut, Edit, Phone, MapPin, CheckSquare,
Sparkles, Brain, Send, AlertTriangle, Banknote,
ShoppingBag, Monitor, Minus, Calculator, UserCog, Percent, ArrowDownLeft
} from ‘lucide-react’;

const APP_NAME = ‘Pro ERP’;
const APP_LOGO = ‘P’;
const CURRENCY = ‘ر.س’;
const DEV_NAME = ‘محمد سعيد’;
const VAT_RATE_DEFAULT = 15;

// ══ TYPES ══
interface User { id:number; name:string; role:‘admin’|‘manager’|‘staff’; email:string; branch:string; active:boolean; lastLogin:string; avatar:string; }
interface Customer { id:number; name:string; phone:string; email:string; address:string; balance:number; totalPurchases:number; type:‘retail’|‘wholesale’; notes:string; }
interface Supplier { id:number; name:string; phone:string; email:string; address:string; balance:number; totalOrders:number; category:string; }
interface InvItem { name:string; qty:number; price:number; vatRate:number; }
interface Invoice { id:number; number:string; type:‘sale’|‘purchase’; party:string; partyId:number; date:string; items:InvItem[]; subtotal:number; vatAmount:number; total:number; paid:number; status:‘paid’|‘partial’|‘unpaid’; }
interface SaleItem { id:number; branch:string; date:string; network:number; cash:number; subtotal:number; vatAmount:number; total:number; status:‘completed’|‘pending’|‘cancelled’; }
interface SoldItem { id:number; productId:number; productName:string; barcode:string; branch:string; date:string; qty:number; unit:string; unitPrice:number; vatRate:number; priceAfterVat:number; }
interface PurchaseItem { id:number; supplierId:number; supplierName:string; productId:number; productName:string; date:string; qty:number; unit:string; unitPrice:number; vatRate:number; priceAfterVat:number; totalCost:number; status:‘received’|‘pending’|‘cancelled’; notes:string; }
interface ExpenseItem { id:number; category:string; amount:number; date:string; notes:string; recurring:boolean; }
interface GoalItem { id:number; title:string; target:number; current:number; deadline:string; color:string; category:string; }
interface Notification { id:number; title:string; message:string; time:string; type:‘warning’|‘success’|‘info’|‘error’; read:boolean; }
interface Product { id:number; name:string; description:string; stock:number; minStock:number; unitPrice:number; vatRate:number; priceAfterVat:number; category:string; barcode:string; unit:string; }
interface Employee { id:number; name:string; position:string; department:string; salary:number; joinDate:string; status:‘active’|‘inactive’; phone:string; email:string; }
interface Payroll { id:number; employeeId:number; employeeName:string; month:string; baseSalary:number; bonus:number; deductions:number; net:number; status:‘paid’|‘pending’; }
interface PosItem { productId:number; name:string; unitPrice:number; vatRate:number; priceAfterVat:number; qty:number; }
interface ChatMsg { role:‘user’|‘assistant’; content:string; }

// ══ UTILS ══
const fmt = (n:number) => `${n.toLocaleString('ar-SA',{minimumFractionDigits:2,maximumFractionDigits:2})} ${CURRENCY}`;
const fmtN = (n:number) => n.toLocaleString(‘ar-SA’,{minimumFractionDigits:2,maximumFractionDigits:2});
const pct = (a:number,b:number) => b>0?Math.min(Math.round((a/b)*100),100):0;
const today = () => new Date().toISOString().split(‘T’)[0];
const calcAfterVat = (p:number,v:number) => p*(1+v/100);
const calcVat = (p:number,v:number) => p*v/100;
const grads = [‘from-blue-500 to-cyan-400’,‘from-violet-500 to-purple-400’,‘from-emerald-500 to-teal-400’,‘from-amber-500 to-orange-400’,‘from-rose-500 to-pink-400’,‘from-indigo-500 to-blue-400’];

// ══ ATOMS ══
const Badge:React.FC<{text:string;color:string}> = ({text,color}) => <span className={`text-xs px-2 py-0.5 rounded-full font-bold whitespace-nowrap ${color}`}>{text}</span>;
const Chip:React.FC<{label:string;value:string;vc?:string}> = ({label,value,vc=‘text-white’}) => (

  <div className="flex justify-between items-center p-3 rounded-xl bg-slate-800/40 border border-slate-700/30">
    <span className="text-xs text-slate-400 font-semibold">{label}</span>
    <span className={`font-black text-sm ${vc}`}>{value}</span>
  </div>
);
const GCard:React.FC<{children:React.ReactNode;className?:string;onClick?:()=>void;glow?:string;style?:React.CSSProperties}> = ({children,className='',onClick,glow,style}) => (
  <div onClick={onClick} className={`relative rounded-2xl border border-slate-800/60 overflow-hidden transition-all duration-300 hover:border-slate-700/60 ${onClick?'cursor-pointer':''} ${className}`} style={{background:'rgba(9,14,28,0.94)',...style}}>
    {glow&&<div className={`absolute inset-0 opacity-0 hover:opacity-5 transition-opacity bg-gradient-to-br ${glow}`}/>}
    {children}
  </div>
);
const BarUI:React.FC<{v:number;grad:string;h?:string}> = ({v,grad,h='h-2'}) => (
  <div className={`w-full bg-slate-700/30 rounded-full ${h} overflow-hidden`}>
    <div className={`${h} rounded-full bg-gradient-to-r ${grad} transition-all duration-700`} style={{width:`${Math.min(v,100)}%`}}/>
  </div>
);
const KpiCard:React.FC<{label:string;value:string;sub?:string;icon:React.ElementType;grad:string;trend?:string;trendUp?:boolean;onClick?:()=>void}> =
  ({label,value,sub,icon:Icon,grad,trend,trendUp,onClick}) => (
  <GCard onClick={onClick} glow={grad} className="p-5 group">
    <div className={`absolute inset-0 opacity-[0.04] group-hover:opacity-[0.07] transition-opacity bg-gradient-to-br ${grad}`}/>
    <div className="relative flex justify-between items-start mb-3">
      <div className={`p-2.5 rounded-xl bg-gradient-to-br ${grad} shadow-lg`}><Icon size={18} className="text-white"/></div>
      {trend&&<span className={`text-xs font-bold px-2 py-1 rounded-full flex items-center gap-0.5 ${trendUp?'bg-emerald-500/15 text-emerald-400':'bg-rose-500/15 text-rose-400'}`}>
        {trendUp?<ArrowUpRight size={10}/>:<ArrowDownRight size={10}/>}{trend}
      </span>}
    </div>
    <p className="text-slate-500 text-xs font-semibold uppercase tracking-widest mb-1">{label}</p>
    <h3 className="text-xl font-black text-white tracking-tight leading-tight">{value}</h3>
    {sub&&<p className="text-slate-600 text-xs mt-1">{sub}</p>}
  </GCard>
);
const EditCell:React.FC<{value:string;onChange:(v:string)=>void;className?:string}> = ({value:init,onChange,className}) => {
  const [val,setVal]=useState(init);const [ed,setEd]=useState(false);
  useEffect(()=>setVal(init),[init]);
  const commit=()=>{setEd(false);onChange(val);};
  return <td onDoubleClick={()=>setEd(true)} className={`border-b border-slate-800/40 p-3 cursor-pointer hover:bg-slate-800/20 transition-colors text-sm ${className}`}>
    {ed?<input autoFocus value={val} onChange={e=>setVal(e.target.value)} onBlur={commit} onKeyDown={e=>e.key==='Enter'&&commit()} className="w-full bg-slate-700 text-white border border-blue-500 rounded-lg px-2 py-1 outline-none text-right"/>:<span>{val}</span>}
  </td>;
};
const Modal:React.FC<{title:string;onClose:()=>void;children:React.ReactNode;wide?:boolean;icon?:React.ElementType;ic?:string}> =
  ({title,onClose,children,wide,icon:Icon,ic='text-blue-400'}) => (
  <div className="fixed inset-0 z-50 flex items-center justify-center p-4" style={{background:'rgba(0,0,0,0.85)',backdropFilter:'blur(14px)'}}>
    <div className={`w-full ${wide?'max-w-3xl':'max-w-lg'} rounded-3xl border border-slate-700/40 shadow-2xl overflow-hidden max-h-[92vh] flex flex-col`} style={{background:'rgba(7,11,24,0.99)'}}>
      <div className="flex justify-between items-center p-5 border-b border-slate-700/40 flex-shrink-0">
        <div className="flex items-center gap-3">
          {Icon&&<div className={`p-2 rounded-xl bg-slate-800/80 ${ic}`}><Icon size={16}/></div>}
          <h3 className="font-black text-white">{title}</h3>
        </div>
        <button onClick={onClose} className="p-1.5 hover:bg-slate-700/60 rounded-xl transition-colors"><X size={16} className="text-slate-400"/></button>
      </div>
      <div className="p-5 overflow-y-auto flex-1">{children}</div>
    </div>
  </div>
);
const Field:React.FC<{label:string;children:React.ReactNode;req?:boolean}> = ({label,children,req}) => (
  <div><label className="text-xs font-bold text-slate-400 uppercase tracking-wider mb-1.5 block">{label}{req&&<span className="text-rose-400 mr-1">*</span>}</label>{children}</div>
);
const Inp:React.FC<{value:string;onChange:(v:string)=>void;type?:string;ph?:string;ac?:string}> = ({value,onChange,type='text',ph,ac='blue'}) => (
  <input type={type} value={value} onChange={e=>onChange(e.target.value)} placeholder={ph}
    className={`w-full bg-slate-800/70 border border-slate-700/50 rounded-xl px-3.5 py-2.5 text-white text-sm outline-none focus:border-${ac}-500 transition-colors text-right placeholder-slate-600`}/>
);
const Sel:React.FC<{value:string;onChange:(v:string)=>void;children:React.ReactNode}> = ({value,onChange,children}) => (
  <select value={value} onChange={e=>onChange(e.target.value)} className="w-full bg-slate-800/70 border border-slate-700/50 rounded-xl px-3.5 py-2.5 text-white text-sm outline-none focus:border-blue-500 transition-colors text-right">{children}</select>
);
const BtnP:React.FC<{onClick:()=>void;children:React.ReactNode;g:string;disabled?:boolean;full?:boolean}> = ({onClick,children,g,disabled,full}) => (
  <button onClick={onClick} disabled={disabled} className={`${full?'w-full':'flex-1'} text-white py-2.5 rounded-xl font-bold text-sm hover:opacity-90 transition-all disabled:opacity-40 flex items-center justify-center gap-2`} style={{background:`linear-gradient(135deg,${g})`}}>{children}</button>
);
const SubTabs:React.FC<{tabs:[string,string][];active:string;onChange:(v:string)=>void}> = ({tabs,active,onChange}) => (
  <div className="flex gap-1 p-1 rounded-xl border border-slate-700/40 w-fit" style={{background:'rgba(12,20,40,0.8)'}}>
    {tabs.map(([k,l])=>(
      <button key={k} onClick={()=>onChange(k)}
        className={`px-4 py-2 rounded-lg text-sm font-bold transition-all ${active===k?'text-white':'text-slate-500 hover:text-slate-300'}`}
        style={active===k?{background:'linear-gradient(135deg,rgba(29,78,216,0.3),rgba(5,150,105,0.2))',boxShadow:'inset 0 0 0 1px rgba(29,78,216,0.3)'}:{}}>
        {l}
      </button>
    ))}
  </div>
);
const EmptyState:React.FC<{icon:React.ElementType;msg:string;sub?:string;action?:()=>void;actionLabel?:string}> = ({icon:Icon,msg,sub,action,actionLabel}) => (
  <GCard className="p-14 text-center"><Icon size={36} className="text-slate-700 mx-auto mb-3"/><p className="text-slate-500 font-semibold">{msg}</p>{sub&&<p className="text-slate-700 text-xs mt-1">{sub}</p>}{action&&<button onClick={action} className="mt-4 text-blue-400 text-sm font-bold hover:text-blue-300">+ {actionLabel}</button>}</GCard>
);

// ══ LOGIN ══
const LoginScreen:React.FC<{onLogin:(u:User)=>void}> = ({onLogin}) => {
const [email,setEmail]=useState(’’);const [pass,setPass]=useState(’’);const [err,setErr]=useState(’’);const [loading,setLoading]=useState(false);
const DEMO:(User&{password:string})[]=[
{id:1,name:‘محمد سعيد’,role:‘admin’,email:‘admin@proerp.com’,password:‘1234’,branch:‘الإدارة العامة’,active:true,lastLogin:‘الآن’,avatar:‘م’},
{id:2,name:‘أحمد علي’,role:‘manager’,email:‘manager@proerp.com’,password:‘1234’,branch:‘الرياض’,active:true,lastLogin:‘أمس’,avatar:‘أ’},
{id:3,name:‘سارة محمود’,role:‘staff’,email:‘staff@proerp.com’,password:‘1234’,branch:‘جدة’,active:true,lastLogin:‘منذ ٣ أيام’,avatar:‘س’},
];
const handle=async()=>{setLoading(true);await new Promise(r=>setTimeout(r,700));const u=DEMO.find(u=>u.email===email&&u.password===pass);if(u){const{password,…usr}=u;onLogin(usr);}else{setErr(‘بيانات الدخول غير صحيحة’);setLoading(false);}};
return(
<div className=“min-h-screen flex items-center justify-center p-4” style={{background:’#040810’,fontFamily:”‘Cairo’,‘Segoe UI’,sans-serif”}}>
<div className="absolute inset-0 pointer-events-none">
<div className=“absolute top-1/3 right-1/4 w-[500px] h-[500px] rounded-full opacity-[0.04]” style={{background:‘radial-gradient(circle,#1d4ed8,transparent)’}}/>
<div className=“absolute bottom-1/3 left-1/4 w-[400px] h-[400px] rounded-full opacity-[0.04]” style={{background:‘radial-gradient(circle,#059669,transparent)’}}/>
</div>
<div className="w-full max-w-md relative z-10">
<div className="text-center mb-8">
<div className="relative inline-block mb-5">
<div className=“w-20 h-20 rounded-2xl flex items-center justify-center font-black text-4xl text-white mx-auto” style={{background:‘linear-gradient(135deg,#1d4ed8,#059669)’,boxShadow:‘0 0 50px rgba(29,78,216,0.4),0 20px 40px rgba(0,0,0,0.4)’}}>P</div>
<div className=“absolute inset-0 rounded-2xl blur-xl opacity-40” style={{background:‘linear-gradient(135deg,#1d4ed8,#059669)’}}/>
</div>
<h1 className="text-3xl font-black text-white mb-1">{APP_NAME}</h1>
<p className="text-slate-500 text-sm">نظام تخطيط موارد المؤسسات</p>
</div>
<div className=“rounded-3xl border border-slate-700/30 p-7 space-y-4” style={{background:‘rgba(9,14,28,0.95)’,boxShadow:‘0 40px 80px rgba(0,0,0,0.5)’}}>
<Field label="البريد الإلكتروني"><Inp value={email} onChange={setEmail} type="email" ph="admin@proerp.com"/></Field>
<Field label="كلمة المرور"><Inp value={pass} onChange={setPass} type="password" ph="••••••••"/></Field>
{err&&<div className="flex items-center gap-2 text-rose-400 text-xs bg-rose-500/8 border border-rose-500/20 rounded-xl px-3 py-2.5"><AlertTriangle size={13}/>{err}</div>}
<button onClick={handle} disabled={loading} className=“w-full text-white py-3 rounded-xl font-black text-sm hover:opacity-90 disabled:opacity-60 flex items-center justify-center gap-2” style={{background:‘linear-gradient(135deg,#1d4ed8,#059669)’,boxShadow:‘0 8px 25px rgba(29,78,216,0.35)’}}>
{loading?<RefreshCw size={16} className="animate-spin"/>:<><LogIn size={16}/>دخول</>}
</button>
<div className="border-t border-slate-700/40 pt-4">
<p className="text-slate-600 text-xs text-center mb-3">حسابات تجريبية · كلمة المرور: 1234</p>
<div className="grid grid-cols-3 gap-2">
{[[‘مدير عام’,‘admin@proerp.com’,‘م’,‘from-blue-600 to-green-500’],[‘مدير’,‘manager@proerp.com’,‘أ’,‘from-blue-500 to-cyan-400’],[‘موظف’,‘staff@proerp.com’,‘س’,‘from-slate-600 to-slate-500’]].map(([l,em,av,g])=>(
<button key={em} onClick={()=>{setEmail(em);setPass(‘1234’);}} className=“flex flex-col items-center py-3 rounded-xl border border-slate-700/50 hover:border-slate-500 hover:bg-slate-800/40 transition-all”>
<div className={`w-8 h-8 rounded-lg flex items-center justify-center text-white font-black text-sm mb-1.5 bg-gradient-to-br ${g}`}>{av}</div>
<span className="text-xs text-slate-400">{l}</span>
</button>
))}
</div>
</div>
</div>
<p className="text-center text-slate-700 text-xs mt-5">تطوير: <span className="text-slate-500 font-bold">{DEV_NAME}</span> · {APP_NAME} v4.1</p>
</div>
</div>
);
};

// ══════════════════════════════════════
// MAIN APP
// ══════════════════════════════════════
export default function App() {
const [user,setUser]=useState<User|null>(null);
const [tab,setTab]=useState(‘dashboard’);
const [search,setSearch]=useState(’’);
const [time,setTime]=useState(new Date());
const [collapsed,setCollapsed]=useState(false);
const [modal,setModal]=useState<string|null>(null);
const [editTarget,setEditTarget]=useState<any>(null);
const [form,setForm]=useState<Record<string,any>>({});
const [invItems,setInvItems]=useState<InvItem[]>([{name:’’,qty:1,price:0,vatRate:VAT_RATE_DEFAULT}]);
const [aiLoading,setAiLoading]=useState(false);
const [aiInsight,setAiInsight]=useState(’’);
const [ocrFile,setOcrFile]=useState<string|null>(null);
const [ocrResult,setOcrResult]=useState(’’);
const [ocrLoading,setOcrLoading]=useState(false);
const [chatOpen,setChatOpen]=useState(false);
const [chatMsgs,setChatMsgs]=useState<ChatMsg[]>([{role:‘assistant’,content:`مرحباً! أنا مساعد ${APP_NAME} 🤖\nيمكنني تحليل بياناتك والإجابة على أسئلتك.`}]);
const [chatInput,setChatInput]=useState(’’);
const [chatLoading,setChatLoading]=useState(false);
const chatEndRef=useRef<HTMLDivElement>(null);
const [posCart,setPosCart]=useState<PosItem[]>([]);
const [posSearch,setPosSearch]=useState(’’);
const [posDiscount,setPosDiscount]=useState(0);
const [posMsg,setPosMsg]=useState(’’);
const [salesSubTab,setSalesSubTab]=useState<‘branches’|‘items’>(‘branches’);
const [soldItems,setSoldItems]=useState<SoldItem[]>([]);
const [soldItemForm,setSoldItemForm]=useState({productId:’’,branch:’’,date:today(),qty:‘1’,customBranch:’’});
const [purchSubTab,setPurchSubTab]=useState<‘list’|‘add’>(‘list’);
const [purchases,setPurchases]=useState<PurchaseItem[]>([]);
const [purchForm,setPurchForm]=useState({supplierId:’’,productId:’’,date:today(),qty:‘1’,notes:’’,customUnitPrice:’’});

useEffect(()=>{const t=setInterval(()=>setTime(new Date()),1000);return()=>clearInterval(t);},[]);
useEffect(()=>{chatEndRef.current?.scrollIntoView({behavior:‘smooth’});},[chatMsgs]);

// ══ DATA ══
const [sales,setSales]=useState<SaleItem[]>([
{id:1,branch:‘الرياض’,date:‘٢٠٢٥-٠٣-٠١’,network:18500,cash:9200,subtotal:23913,vatAmount:3587,total:27500,status:‘completed’},
{id:2,branch:‘جدة’,date:‘٢٠٢٥-٠٣-٠٢’,network:14200,cash:7100,subtotal:18522,vatAmount:2778,total:21300,status:‘completed’},
{id:3,branch:‘الدمام’,date:‘٢٠٢٥-٠٣-٠٣’,network:22400,cash:11300,subtotal:29304,vatAmount:4396,total:33700,status:‘pending’},
{id:4,branch:‘مكة’,date:‘٢٠٢٥-٠٣-٠٤’,network:16800,cash:8400,subtotal:21913,vatAmount:3287,total:25200,status:‘completed’},
]);
const [expenses,setExpenses]=useState<ExpenseItem[]>([
{id:1,category:‘إيجار’,amount:5000,date:‘٢٠٢٥-٠٣-٠١’,notes:‘إيجار مكتب الرياض’,recurring:true},
{id:2,category:‘كهرباء’,amount:1200,date:‘٢٠٢٥-٠٣-٠٥’,notes:‘فاتورة الكهرباء’,recurring:false},
{id:3,category:‘رواتب’,amount:38000,date:‘٢٠٢٥-٠٣-٠١’,notes:‘رواتب الموظفين’,recurring:true},
{id:4,category:‘تسويق’,amount:3500,date:‘٢٠٢٥-٠٣-٠٨’,notes:‘إعلانات رقمية’,recurring:false},
]);
const [goals,setGoals]=useState<GoalItem[]>([
{id:1,title:‘هدف المبيعات الشهري’,target:150000,current:124700,deadline:‘٣١ مارس’,color:‘from-blue-500 to-cyan-400’,category:‘مبيعات’},
{id:2,title:‘تخفيض المصروفات’,target:50000,current:47700,deadline:‘٣١ مارس’,color:‘from-emerald-500 to-teal-400’,category:‘مالية’},
{id:3,title:‘هدف صافي الربح’,target:80000,current:52300,deadline:‘٣١ مارس’,color:‘from-violet-500 to-purple-400’,category:‘مالية’},
{id:4,title:‘افتتاح فروع جديدة’,target:7,current:5,deadline:‘٣١ ديسمبر’,color:‘from-amber-500 to-orange-400’,category:‘توسع’},
]);
const [products,setProducts]=useState<Product[]>([
{id:1,name:‘جهاز لابتوب ProBook’,description:‘لابتوب للأعمال — معالج i7، رام 16GB، SSD 512GB’,stock:5,minStock:10,unitPrice:2608.70,vatRate:15,priceAfterVat:2999,category:‘إلكترونيات’,barcode:‘BAR-001’,unit:‘جهاز’},
{id:2,name:‘سماعات بلوتوث Elite’,description:‘سماعات لاسلكية — إلغاء ضجيف، بطارية 30 ساعة’,stock:23,minStock:15,unitPrice:303.48,vatRate:15,priceAfterVat:349,category:‘إكسسوارات’,barcode:‘BAR-002’,unit:‘قطعة’},
{id:3,name:‘شاشة 4K UltraWide’,description:‘شاشة 32 بوصة — 4K UHD، HDR، 144Hz’,stock:2,minStock:8,unitPrice:1390.43,vatRate:15,priceAfterVat:1599,category:‘إلكترونيات’,barcode:‘BAR-003’,unit:‘جهاز’},
{id:4,name:‘كيبورد لاسلكي Slim’,description:‘كيبورد عربي/إنجليزي — رفيع، بلوتوث’,stock:41,minStock:20,unitPrice:164.35,vatRate:15,priceAfterVat:189,category:‘مستلزمات’,barcode:‘BAR-004’,unit:‘قطعة’},
{id:5,name:‘ماوس ألعاب RGB’,description:‘ماوس ألعاب — DPI قابل للضبط، إضاءة RGB’,stock:17,minStock:10,unitPrice:216.52,vatRate:15,priceAfterVat:249,category:‘مستلزمات’,barcode:‘BAR-005’,unit:‘قطعة’},
]);
const [notifications,setNotifications]=useState<Notification[]>([
{id:1,title:‘مخزون منخفض’,message:‘جهاز لابتوب — ٥ وحدات’,time:‘منذ ٣٠ دقيقة’,type:‘warning’,read:false},
{id:2,title:‘تم تجاوز الهدف!’,message:‘تحقق ١١٢٪ من هدف اليوم’,time:‘منذ ساعة’,type:‘success’,read:false},
{id:3,title:‘فواتير معلقة’,message:‘فواتير بقيمة ٢٣,٧٠٠ ر.س’,time:‘منذ ساعتين’,type:‘error’,read:false},
{id:4,title:‘التقرير الأسبوعي’,message:‘تقرير الأداء جاهز’,time:‘منذ ٣ ساعات’,type:‘info’,read:true},
]);
const [users,setUsers]=useState<User[]>([
{id:1,name:‘محمد سعيد’,role:‘admin’,email:‘admin@proerp.com’,branch:‘الإدارة العامة’,active:true,lastLogin:‘الآن’,avatar:‘م’},
{id:2,name:‘أحمد علي’,role:‘manager’,email:‘manager@proerp.com’,branch:‘الرياض’,active:true,lastLogin:‘أمس’,avatar:‘أ’},
{id:3,name:‘سارة محمود’,role:‘staff’,email:‘staff@proerp.com’,branch:‘جدة’,active:true,lastLogin:‘منذ ٣ أيام’,avatar:‘س’},
]);
const [customers,setCustomers]=useState<Customer[]>([
{id:1,name:‘شركة الخليج للتجارة’,phone:‘0500-123-456’,email:‘gulf@trade.com’,address:‘الرياض’,balance:12500,totalPurchases:145000,type:‘wholesale’,notes:‘عميل مميز’},
{id:2,name:‘أحمد محمد العمري’,phone:‘0512-345-678’,email:‘ahmed@mail.com’,address:‘جدة’,balance:0,totalPurchases:8700,type:‘retail’,notes:’’},
{id:3,name:‘مؤسسة الأمانة’,phone:‘0534-222-333’,email:‘amanah@corp.com’,address:‘الدمام’,balance:5200,totalPurchases:67000,type:‘wholesale’,notes:’’},
]);
const [suppliers,setSuppliers]=useState<Supplier[]>([
{id:1,name:‘شركة سامسونج السعودية’,phone:‘011-222-3333’,email:‘samsung@sa.com’,address:‘الرياض’,balance:8000,totalOrders:95000,category:‘إلكترونيات’},
{id:2,name:‘مستودعات الخليج’,phone:‘012-333-4444’,email:‘gulf@warehouse.com’,address:‘جدة’,balance:15000,totalOrders:230000,category:‘مستلزمات’},
]);
const [invoices,setInvoices]=useState<Invoice[]>([
{id:1,number:‘INV-001’,type:‘sale’,party:‘شركة الخليج للتجارة’,partyId:1,date:‘٢٠٢٥-٠٣-٠١’,items:[{name:‘لابتوب’,qty:5,price:2608.70,vatRate:15}],subtotal:13043.50,vatAmount:1956.53,total:15000.03,paid:15000.03,status:‘paid’},
{id:2,number:‘INV-002’,type:‘sale’,party:‘أحمد محمد العمري’,partyId:2,date:‘٢٠٢٥-٠٣-٠٣’,items:[{name:‘شاشة 4K’,qty:1,price:1390.43,vatRate:15}],subtotal:1390.43,vatAmount:208.56,total:1598.99,paid:800,status:‘partial’},
{id:3,number:‘INV-003’,type:‘purchase’,party:‘شركة سامسونج السعودية’,partyId:1,date:‘٢٠٢٥-٠٣-٠٥’,items:[{name:‘لابتوب x20’,qty:20,price:1913.04,vatRate:15}],subtotal:38260.87,vatAmount:5739.13,total:44000,paid:0,status:‘unpaid’},
]);
const [employees,setEmployees]=useState<Employee[]>([
{id:1,name:‘أحمد علي المطيري’,position:‘مدير مبيعات’,department:‘المبيعات’,salary:8000,joinDate:‘٢٠٢٢-٠١-١٥’,status:‘active’,phone:‘0501-111-222’,email:‘ahmed@proerp.com’},
{id:2,name:‘سارة محمود الغامدي’,position:‘محاسبة’,department:‘المالية’,salary:6500,joinDate:‘٢٠٢٣-٠٣-٠١’,status:‘active’,phone:‘0512-333-444’,email:‘sara@proerp.com’},
{id:3,name:‘خالد عمر الشمري’,position:‘مندوب مبيعات’,department:‘المبيعات’,salary:5000,joinDate:‘٢٠٢٣-٠٦-١٥’,status:‘active’,phone:‘0523-555-666’,email:‘khalid@proerp.com’},
{id:4,name:‘نورة فهد السعيد’,position:‘موارد بشرية’,department:‘الموارد البشرية’,salary:7000,joinDate:‘٢٠٢٤-٠١-٠١’,status:‘active’,phone:‘0534-777-888’,email:‘noura@proerp.com’},
]);
const [payrolls,setPayrolls]=useState<Payroll[]>([
{id:1,employeeId:1,employeeName:‘أحمد علي المطيري’,month:‘مارس ٢٠٢٥’,baseSalary:8000,bonus:1000,deductions:200,net:8800,status:‘paid’},
{id:2,employeeId:2,employeeName:‘سارة محمود الغامدي’,month:‘مارس ٢٠٢٥’,baseSalary:6500,bonus:500,deductions:0,net:7000,status:‘pending’},
{id:3,employeeId:3,employeeName:‘خالد عمر الشمري’,month:‘مارس ٢٠٢٥’,baseSalary:5000,bonus:750,deductions:100,net:5650,status:‘pending’},
{id:4,employeeId:4,employeeName:‘نورة فهد السعيد’,month:‘مارس ٢٠٢٥’,baseSalary:7000,bonus:0,deductions:0,net:7000,status:‘paid’},
]);

// ══ COMPUTED ══
const activeSales=sales.filter(s=>s.status!==‘cancelled’);
const totalSales=activeSales.reduce((a,s)=>a+s.total,0);
const totalSalesSubtotal=activeSales.reduce((a,s)=>a+s.subtotal,0);
const totalSalesVat=activeSales.reduce((a,s)=>a+s.vatAmount,0);
const totalExpenses=expenses.reduce((a,e)=>a+e.amount,0);
const netProfit=totalSales-totalExpenses;
const unread=notifications.filter(n=>!n.read).length;
const lowStock=products.filter(p=>p.stock<=p.minStock);
const unpaidInvs=invoices.filter(i=>i.status!==‘paid’);
const totalReceivables=invoices.filter(i=>i.type===‘sale’).reduce((a,i)=>a+(i.total-i.paid),0);
const totalPayables=invoices.filter(i=>i.type===‘purchase’).reduce((a,i)=>a+(i.total-i.paid),0);
const totalPayroll=payrolls.reduce((a,p)=>a+p.net,0);
const pendingPayroll=payrolls.filter(p=>p.status===‘pending’).reduce((a,p)=>a+p.net,0);
const totalPurchases=purchases.filter(p=>p.status===‘received’).reduce((a,p)=>a+p.totalCost,0);
const pendingPurchasesCount=purchases.filter(p=>p.status===‘pending’).length;
const posSubtotal=posCart.reduce((a,i)=>a+(i.unitPrice*i.qty),0);
const posVatAmt=posCart.reduce((a,i)=>a+(calcVat(i.unitPrice,i.vatRate)*i.qty),0);
const posTotal=posSubtotal+posVatAmt;
const posDiscountAmt=posTotal*(posDiscount/100);
const posFinal=posTotal-posDiscountAmt;
const branchList=[…new Set(sales.map(s=>s.branch))];
const totalInventoryValue=products.reduce((a,p)=>a+(p.priceAfterVat*p.stock),0);

// ══ HANDLERS ══
const openModal=(name:string,target?:any)=>{setModal(name);setEditTarget(target||null);setForm(target?{…target}:{vatRate:String(VAT_RATE_DEFAULT)});setInvItems([{name:’’,qty:1,price:0,vatRate:VAT_RATE_DEFAULT}]);};
const closeModal=()=>{setModal(null);setEditTarget(null);setForm({});};

const addSoldItem=()=>{
const p=products.find(x=>x.id===parseInt(soldItemForm.productId));
if(!p)return;
const qty=parseFloat(soldItemForm.qty)||0;
if(qty<=0||qty>p.stock)return;
const branch=soldItemForm.branch===’**new**’?soldItemForm.customBranch:soldItemForm.branch;
if(!branch)return;
setProducts(prev=>prev.map(x=>x.id===p.id?{…x,stock:x.stock-qty}:x));
setSoldItems(prev=>[{id:Date.now(),productId:p.id,productName:p.name,barcode:p.barcode,branch,date:soldItemForm.date,qty,unit:p.unit,unitPrice:p.unitPrice,vatRate:p.vatRate,priceAfterVat:p.priceAfterVat},…prev]);
setSoldItemForm(f=>({…f,productId:’’,qty:‘1’,customBranch:’’,branch:’’}));
};

const addPurchase=()=>{
const p=products.find(x=>x.id===parseInt(purchForm.productId));
const s=suppliers.find(x=>x.id===parseInt(purchForm.supplierId));
if(!p||!s)return;
const qty=parseFloat(purchForm.qty)||0;
if(qty<=0)return;
const unitPrice=purchForm.customUnitPrice?parseFloat(purchForm.customUnitPrice):p.unitPrice;
const priceAV=calcAfterVat(unitPrice,p.vatRate);
const totalCost=priceAV*qty;
setPurchases(prev=>[{id:Date.now(),supplierId:s.id,supplierName:s.name,productId:p.id,productName:p.name,date:purchForm.date,qty,unit:p.unit,unitPrice,vatRate:p.vatRate,priceAfterVat:priceAV,totalCost,status:‘received’,notes:purchForm.notes},…prev]);
setProducts(prev=>prev.map(x=>x.id===p.id?{…x,stock:x.stock+qty}:x));
setSuppliers(prev=>prev.map(x=>x.id===s.id?{…x,totalOrders:x.totalOrders+totalCost}:x));
setPurchForm({supplierId:’’,productId:’’,date:today(),qty:‘1’,notes:’’,customUnitPrice:’’});
setPurchSubTab(‘list’);
};

const saveProduct=()=>{
if(!form.name)return;
const unitPrice=parseFloat(form.unitPrice)||0;
const vatRate=parseFloat(form.vatRate)||VAT_RATE_DEFAULT;
const priceAfterVat=parseFloat(form.priceAfterVat)||calcAfterVat(unitPrice,vatRate);
const p:Product={id:editTarget?editTarget.id:Date.now(),name:form.name,description:form.description||’’,stock:parseInt(form.stock)||0,minStock:parseInt(form.minStock)||5,unitPrice,vatRate,priceAfterVat,category:form.category||‘عام’,barcode:form.barcode||`BAR-${Date.now()}`,unit:form.unit||‘قطعة’};
if(editTarget)setProducts(prev=>prev.map(x=>x.id===editTarget.id?p:x));else setProducts(prev=>[p,…prev]);
closeModal();
};
const saveUser=()=>{if(!form.name||!form.email)return;if(editTarget)setUsers(u=>u.map(x=>x.id===editTarget.id?{…x,…form,avatar:form.name.charAt(0)}:x));else setUsers(u=>[{id:Date.now(),name:form.name,role:form.role||‘staff’,email:form.email,branch:form.branch||’’,active:true,lastLogin:‘الآن’,avatar:form.name.charAt(0)},…u]);closeModal();};
const saveCustomer=()=>{if(!form.name)return;if(editTarget)setCustomers(c=>c.map(x=>x.id===editTarget.id?{…x,…form}:x));else setCustomers(c=>[{id:Date.now(),name:form.name,phone:form.phone||’’,email:form.email||’’,address:form.address||’’,balance:0,totalPurchases:0,type:form.type||‘retail’,notes:form.notes||’’},…c]);closeModal();};
const saveSupplier=()=>{if(!form.name)return;if(editTarget)setSuppliers(s=>s.map(x=>x.id===editTarget.id?{…x,…form}:x));else setSuppliers(s=>[{id:Date.now(),name:form.name,phone:form.phone||’’,email:form.email||’’,address:form.address||’’,balance:0,totalOrders:0,category:form.category||’’},…s]);closeModal();};
const saveInvoice=()=>{
if(!form.party||invItems.every(i=>!i.name))return;
const subtotal=invItems.reduce((a,i)=>a+(i.price*i.qty),0);
const vatAmount=invItems.reduce((a,i)=>a+(calcVat(i.price,i.vatRate)*i.qty),0);
setInvoices(inv=>[{id:Date.now(),number:`INV-${String(invoices.length+1).padStart(3,'0')}`,type:form.type||‘sale’,party:form.party,partyId:0,date:form.date||today(),items:invItems.filter(i=>i.name),subtotal,vatAmount,total:subtotal+vatAmount,paid:0,status:‘unpaid’},…inv]);
closeModal();
};
const saveEmployee=()=>{if(!form.name)return;if(editTarget)setEmployees(e=>e.map(x=>x.id===editTarget.id?{…x,…form}:x));else setEmployees(e=>[{id:Date.now(),name:form.name,position:form.position||’’,department:form.department||’’,salary:parseFloat(form.salary)||0,joinDate:form.joinDate||today(),status:‘active’,phone:form.phone||’’,email:form.email||’’},…e]);closeModal();};
const recordPayment=(invId:number,amount:number)=>{setInvoices(p=>p.map(i=>{if(i.id!==invId)return i;const np=Math.min(i.paid+amount,i.total);return{…i,paid:np,status:np>=i.total?‘paid’:np>0?‘partial’:‘unpaid’};}));};
const payPayroll=(id:number)=>setPayrolls(p=>p.map(x=>x.id===id?{…x,status:‘paid’}:x));
const generatePayroll=()=>{const m=‘أبريل ٢٠٢٥’;employees.filter(e=>e.status===‘active’).forEach(emp=>{if(!payrolls.find(p=>p.employeeId===emp.id&&p.month===m)){setPayrolls(p=>[…p,{id:Date.now()+emp.id,employeeId:emp.id,employeeName:emp.name,month:m,baseSalary:emp.salary,bonus:0,deductions:0,net:emp.salary,status:‘pending’}]);}});};
const addToCart=(p:Product)=>{if(p.stock<=0)return;setPosCart(c=>{const ex=c.find(x=>x.productId===p.id);if(ex)return c.map(x=>x.productId===p.id?{…x,qty:x.qty+1}:x);return[…c,{productId:p.id,name:p.name,unitPrice:p.unitPrice,vatRate:p.vatRate,priceAfterVat:p.priceAfterVat,qty:1}];});};
const removeFromCart=(id:number)=>setPosCart(c=>c.filter(x=>x.productId!==id));
const updateQty=(id:number,qty:number)=>setPosCart(c=>c.map(x=>x.productId===id?{…x,qty:Math.max(1,qty)}:x));
const checkout=()=>{
if(posCart.length===0)return;
const sub=posCart.reduce((a,i)=>a+(i.unitPrice*i.qty),0);
const vat=posCart.reduce((a,i)=>a+(calcVat(i.unitPrice,i.vatRate)*i.qty),0);
const tot=(sub+vat)*(1-posDiscount/100);
setInvoices(prev=>[{id:Date.now(),number:`POS-${String(invoices.length+1).padStart(3,'0')}`,type:‘sale’,party:‘نقطة البيع’,partyId:0,date:today(),items:posCart.map(i=>({name:i.name,qty:i.qty,price:i.unitPrice,vatRate:i.vatRate})),subtotal:sub*(1-posDiscount/100),vatAmount:vat*(1-posDiscount/100),total:tot,paid:tot,status:‘paid’},…prev]);
posCart.forEach(ci=>setProducts(p=>p.map(x=>x.id===ci.productId?{…x,stock:Math.max(0,x.stock-ci.qty)}:x)));
setPosCart([]);setPosDiscount(0);
setPosMsg(`✓ تمت عملية البيع — ${fmt(tot)}`);setTimeout(()=>setPosMsg(’’),4000);
};

const getAiInsight=async()=>{setAiLoading(true);setAiInsight(’’);try{const r=await fetch(‘https://api.anthropic.com/v1/messages’,{method:‘POST’,headers:{‘Content-Type’:‘application/json’},body:JSON.stringify({model:‘claude-sonnet-4-20250514’,max_tokens:800,messages:[{role:‘user’,content:`أنت محلل أعمال. حلل وقدم ٤ توصيات مختصرة:\nمبيعات:${totalSales}, مشتريات:${totalPurchases}, مصروفات:${totalExpenses}, أرباح:${netProfit}, ضريبة:${totalSalesVat}, موظفون:${employees.length}, مخزون منخفض:${lowStock.length}`}]})}); const d=await r.json();setAiInsight(d.content?.map((c:any)=>c.type===‘text’?c.text:’’).join(’’)||‘تعذر التوليد.’);}catch{setAiInsight(‘خطأ في الاتصال.’);}setAiLoading(false);};
const sendChat=async()=>{if(!chatInput.trim()||chatLoading)return;const msg=chatInput.trim();setChatInput(’’);setChatMsgs(m=>[…m,{role:‘user’,content:msg}]);setChatLoading(true);try{const ctx=`مساعد ${APP_NAME}. البيانات: مبيعات=${fmt(totalSales)}, مشتريات=${fmt(totalPurchases)}, أرباح=${fmt(netProfit)}, ضريبة=${fmt(totalSalesVat)}, موظفون=${employees.length}. أجب بالعربية مختصراً.`;const r=await fetch(‘https://api.anthropic.com/v1/messages’,{method:‘POST’,headers:{‘Content-Type’:‘application/json’},body:JSON.stringify({model:‘claude-sonnet-4-20250514’,max_tokens:500,system:ctx,messages:[…chatMsgs.slice(-6).map(m=>({role:m.role,content:m.content})),{role:‘user’,content:msg}]})});const d=await r.json();setChatMsgs(m=>[…m,{role:‘assistant’,content:d.content?.map((c:any)=>c.type===‘text’?c.text:’’).join(’’)||‘تعذر الرد.’}]);}catch{setChatMsgs(m=>[…m,{role:‘assistant’,content:‘خطأ، يرجى المحاولة.’}]);}setChatLoading(false);};
const handleOcr=async(e:React.ChangeEvent<HTMLInputElement>)=>{const file=e.target.files?.[0];if(!file)return;setOcrLoading(true);setOcrResult(’’);const reader=new FileReader();reader.onload=async(ev)=>{const b64=(ev.target?.result as string).split(’,’)[1];setOcrFile(ev.target?.result as string);try{const r=await fetch(‘https://api.anthropic.com/v1/messages’,{method:‘POST’,headers:{‘Content-Type’:‘application/json’},body:JSON.stringify({model:‘claude-sonnet-4-20250514’,max_tokens:1000,messages:[{role:‘user’,content:[{type:‘image’,source:{type:‘base64’,media_type:file.type,data:b64}},{type:‘text’,text:‘استخرج بيانات الفاتورة: المورد/العميل، التاريخ، الرقم، البنود، الأسعار قبل وبعد الضريبة، الإجمالي. بالعربية.’}]}]})});const d=await r.json();setOcrResult(d.content?.map((c:any)=>c.type===‘text’?c.text:’’).join(’’)||‘لم يتم الاستخراج.’);}catch{setOcrResult(‘خطأ في المعالجة.’);}setOcrLoading(false);};reader.readAsDataURL(file);};

const printReport=()=>{
const w=window.open(’’,’_blank’);if(!w)return;
const vatColl=invoices.filter(i=>i.type===‘sale’).reduce((a,i)=>a+i.vatAmount,0);
const vatPaid=invoices.filter(i=>i.type===‘purchase’).reduce((a,i)=>a+i.vatAmount,0);
w.document.write(`<!DOCTYPE html><html dir="rtl" lang="ar"><head><meta charset="UTF-8"><title>${APP_NAME}</title> <style>@import url('https://fonts.googleapis.com/css2?family=Cairo:wght@400;700;900&display=swap'); *{margin:0;padding:0;box-sizing:border-box}body{font-family:'Cairo',sans-serif;padding:28px;font-size:12px;color:#1e293b} .h{display:flex;justify-content:space-between;align-items:flex-start;border-bottom:3px solid #1d4ed8;padding-bottom:16px;margin-bottom:20px} .logo{font-size:26px;font-weight:900;color:#1d4ed8} .kpis{display:grid;grid-template-columns:repeat(4,1fr);gap:10px;margin-bottom:20px} .kpi{background:#f8fafc;border:1px solid #e2e8f0;border-radius:10px;padding:12px;text-align:center;border-top:3px solid} .kpi-l{font-size:9px;color:#64748b;text-transform:uppercase;margin-bottom:4px;font-weight:700;letter-spacing:.5px} .kpi-v{font-size:16px;font-weight:900} .sec{margin-bottom:20px}.st{font-size:13px;font-weight:900;margin-bottom:8px;padding:6px 12px;background:#f1f5f9;border-right:4px solid #1d4ed8;border-radius:0 8px 8px 0} table{width:100%;border-collapse:collapse}th{background:#1e293b;color:#fff;padding:7px 9px;text-align:right;font-size:10px;font-weight:700} td{padding:6px 9px;border-bottom:1px solid #f1f5f9;font-size:11px}tr:nth-child(even) td{background:#fafbfc} tfoot td{background:#f1f5f9;font-weight:700} .badge{display:inline-block;padding:1px 7px;border-radius:20px;font-size:10px;font-weight:700} .bg{background:#dcfce7;color:#166534}.by{background:#fef9c3;color:#854d0e}.br{background:#fee2e2;color:#991b1b}.bb{background:#dbeafe;color:#1e40af} .vat-row{display:grid;grid-template-columns:repeat(3,1fr);background:#eff6ff;border:1px solid #bfdbfe;border-radius:10px;padding:14px;margin-bottom:20px;text-align:center} .foot{margin-top:24px;border-top:2px solid #e2e8f0;padding-top:12px;display:flex;justify-content:space-between;color:#94a3b8;font-size:10px} @media print{body{padding:12px}}</style></head><body> <div class="h"><div><div class="logo">${APP_NAME} — تقرير شامل</div><div style="color:#64748b;font-size:11px">نظام تخطيط موارد المؤسسات</div></div> <div style="text-align:left;font-size:11px;color:#64748b"><div>${new Date().toLocaleDateString('ar-SA',{weekday:'long',year:'numeric',month:'long',day:'numeric'})}</div><div>أعده: ${user?.name}</div></div></div> <div class="kpis"> <div class="kpi" style="border-top-color:#1d4ed8"><div class="kpi-l">المبيعات</div><div class="kpi-v" style="color:#1d4ed8">${fmtN(totalSales)} ${CURRENCY}</div></div> <div class="kpi" style="border-top-color:#059669"><div class="kpi-l">المشتريات</div><div class="kpi-v" style="color:#059669">${fmtN(totalPurchases)} ${CURRENCY}</div></div> <div class="kpi" style="border-top-color:#dc2626"><div class="kpi-l">المصروفات</div><div class="kpi-v" style="color:#dc2626">${fmtN(totalExpenses)} ${CURRENCY}</div></div> <div class="kpi" style="border-top-color:#7c3aed"><div class="kpi-l">صافي الربح</div><div class="kpi-v" style="color:#7c3aed">${fmtN(netProfit)} ${CURRENCY}</div></div> </div> <div class="vat-row"> <div><div class="kpi-l">ضريبة محصلة</div><div class="kpi-v" style="color:#059669">${fmtN(vatColl)} ${CURRENCY}</div></div> <div><div class="kpi-l">ضريبة مدفوعة</div><div class="kpi-v" style="color:#dc2626">${fmtN(vatPaid)} ${CURRENCY}</div></div> <div><div class="kpi-l">صافي مستحق</div><div class="kpi-v" style="color:#1d4ed8">${fmtN(vatColl-vatPaid)} ${CURRENCY}</div></div> </div> <div class="sec"><div class="st">🏪 مبيعات الفروع</div><table><thead><tr><th>الفرع</th><th>التاريخ</th><th>قبل الضريبة</th><th>الضريبة</th><th>الإجمالي</th><th>الحالة</th></tr></thead><tbody> ${sales.map(s=>`<tr><td><b>${s.branch}</b></td><td>${s.date}</td><td>${fmtN(s.subtotal)}</td><td style="color:#d97706">${fmtN(s.vatAmount)}</td><td><b>${fmtN(s.total)}</b></td><td><span class="badge ${s.status==='completed'?'bg':s.status==='pending'?'by':'br'}">${s.status===‘completed’?‘مكتمل’:s.status===‘pending’?‘معلق’:‘ملغي’}</span></td></tr>`).join('')} <tfoot><tr><td colspan="2"><b>الإجمالي</b></td><td><b>${fmtN(totalSalesSubtotal)}</b></td><td style="color:#d97706"><b>${fmtN(totalSalesVat)}</b></td><td><b style="color:#1d4ed8">${fmtN(totalSales)} ${CURRENCY}</b></td><td></td></tr></tfoot></table></div> ${soldItems.length>0?`<div class="sec"><div class="st">📦 المنتجات المباعة</div><table><thead><tr><th>المنتج</th><th>الفرع</th><th>الكمية</th><th>سعر الوحدة</th><th>الإجمالي</th></tr></thead><tbody>
${soldItems.map(i=>`<tr><td><b>${i.productName}</b></td><td>${i.branch}</td><td>${i.qty} ${i.unit}</td><td>${fmtN(i.unitPrice)}</td><td><b style="color:#059669">${fmtN(i.priceAfterVat*i.qty)}</b></td></tr>`).join(’’)}
<tfoot><tr><td colspan="4"><b>الإجمالي</b></td><td><b style="color:#059669">${fmtN(soldItems.reduce((a,i)=>a+(i.priceAfterVat*i.qty),0))} ${CURRENCY}</b></td></tr></tfoot></tbody></table></div>`:''} ${purchases.length>0?`<div class="sec"><div class="st">📥 المشتريات</div><table><thead><tr><th>المنتج</th><th>المورد</th><th>الكمية</th><th>سعر الوحدة</th><th>الإجمالي</th><th>الحالة</th></tr></thead><tbody>
${purchases.map(p=>`<tr><td><b>${p.productName}</b></td><td>${p.supplierName}</td><td>${p.qty} ${p.unit}</td><td>${fmtN(p.unitPrice)}</td><td><b>${fmtN(p.totalCost)}</b></td><td><span class="badge ${p.status==='received'?'bg':p.status==='pending'?'by':'br'}">${p.status==='received'?'مستلم':p.status==='pending'?'معلق':'ملغي'}</span></td></tr>`).join(’’)}
<tfoot><tr><td colspan="4"><b>إجمالي المشتريات المستلمة</b></td><td><b style="color:#059669">${fmtN(totalPurchases)} ${CURRENCY}</b></td><td></td></tr></tfoot></tbody></table></div>`:''} <div class="sec"><div class="st">💸 المصروفات</div><table><thead><tr><th>التصنيف</th><th>المبلغ</th><th>التاريخ</th><th>النوع</th></tr></thead><tbody> ${expenses.map(e=>`<tr><td><b>${e.category}</b></td><td style="color:#dc2626;font-weight:700">${fmtN(e.amount)}</td><td>${e.date}</td><td><span class="badge ${e.recurring?'by':'bb'}">${e.recurring?‘متكرر’:‘مرة واحدة’}</span></td></tr>`).join('')} <tfoot><tr><td><b>الإجمالي</b></td><td style="color:#dc2626;font-weight:900">${fmtN(totalExpenses)} ${CURRENCY}</td><td colspan="2"></td></tr></tfoot></table></div> <div class="sec"><div class="st">📦 المخزون</div><table><thead><tr><th>المنتج</th><th>قبل الضريبة</th><th>الضريبة%</th><th>بعد الضريبة</th><th>المخزون</th><th>الحد الأدنى</th></tr></thead><tbody> ${products.map(p=>`<tr><td><b>${p.name}</b></td><td>${fmtN(p.unitPrice)}</td><td>${p.vatRate}٪</td><td style="color:#059669"><b>${fmtN(p.priceAfterVat)}</b></td><td style="color:${p.stock<=p.minStock?'#dc2626':'#059669'};font-weight:700">${p.stock} ${p.unit}</td><td>${p.minStock}</td></tr>`).join('')} </tbody></table></div> <div class="sec"><div class="st">👥 الرواتب</div><table><thead><tr><th>الموظف</th><th>الأساسي</th><th>مكافآت</th><th>خصومات</th><th>الصافي</th><th>الحالة</th></tr></thead><tbody> ${payrolls.map(p=>`<tr><td><b>${p.employeeName}</b></td><td>${fmtN(p.baseSalary)}</td><td style="color:#059669">+${fmtN(p.bonus)}</td><td style="color:#dc2626">-${fmtN(p.deductions)}</td><td><b>${fmtN(p.net)}</b></td><td><span class="badge ${p.status==='paid'?'bg':'by'}">${p.status===‘paid’?‘مدفوع’:‘معلق’}</span></td></tr>`).join('')} <tfoot><tr><td><b>الإجمالي</b></td><td></td><td></td><td></td><td><b style="color:#7c3aed">${fmtN(totalPayroll)} ${CURRENCY}</b></td><td></td></tr></tfoot></table></div> <div class="foot"><div>${APP_NAME} v4.1 · ${new Date().toLocaleDateString('ar-SA')}</div><div style="font-weight:900;color:#1d4ed8">تطوير: ${DEV_NAME}</div></div> <script>window.onload=()=>window.print()</script></body></html>`);w.document.close();
};

const can=(a:string)=>{if(!user)return false;if(user.role===‘admin’)return true;if(user.role===‘manager’)return[‘sales’,‘purchases’,‘expenses’,‘inventory’,‘goals’,‘customers’,‘invoices’,‘analytics’,‘notifications’,‘ocr’,‘reports’,‘payroll’].includes(a);return[‘sales’,‘purchases’,‘inventory’,‘notifications’,‘ocr’,‘pos’].includes(a);};

const navItems=[
{id:‘dashboard’,icon:LayoutDashboard,label:‘الرئيسية’},
{id:‘pos’,icon:Monitor,label:‘نقطة البيع’},
{id:‘sales’,icon:ShoppingCart,label:‘المبيعات’,badge:sales.length,red:false},
{id:‘purchases’,icon:ArrowDownLeft,label:‘المشتريات’,badge:pendingPurchasesCount>0?pendingPurchasesCount:undefined,red:true},
{id:‘expenses’,icon:Wallet,label:‘المصروفات’},
{id:‘inventory’,icon:Package,label:‘المخزون’,badge:lowStock.length>0?lowStock.length:undefined,red:true},
{id:‘goals’,icon:Target,label:‘الأهداف’},
{id:‘customers’,icon:Users,label:‘العملاء والموردون’},
{id:‘invoices’,icon:Receipt,label:‘الفواتير’,badge:unpaidInvs.length>0?unpaidInvs.length:undefined,red:true},
{id:‘payroll’,icon:Banknote,label:‘الرواتب’,badge:payrolls.filter(p=>p.status===‘pending’).length>0?payrolls.filter(p=>p.status===‘pending’).length:undefined,red:false},
{id:‘analytics’,icon:PieChart,label:‘التحليلات’},
{id:‘ocr’,icon:FileText,label:‘فواتير AI’},
{id:‘reports’,icon:Printer,label:‘التقارير’},
…(user?.role===‘admin’?[{id:‘users’,icon:Shield,label:‘المستخدمون’}]:[]),
{id:‘notifications’,icon:Bell,label:‘التنبيهات’,badge:unread>0?unread:undefined,red:true},
].filter(n=>n.id===‘dashboard’||n.id===‘notifications’||(n.id===‘users’&&user?.role===‘admin’)||can(n.id));

const titles:Record<string,string>={dashboard:‘الرئيسية’,pos:‘نقطة البيع’,sales:‘المبيعات’,purchases:‘المشتريات’,expenses:‘المصروفات’,inventory:‘المخزون’,goals:‘الأهداف’,customers:‘العملاء والموردون’,invoices:‘الفواتير’,payroll:‘الرواتب’,analytics:‘التحليلات’,ocr:‘قارئ الفواتير’,reports:‘التقارير’,users:‘المستخدمون’,notifications:‘التنبيهات’};

if(!user)return <LoginScreen onLogin={setUser}/>;

return(
<div className=“flex min-h-screen text-slate-200” dir=“rtl” style={{background:’#040810’,fontFamily:”‘Cairo’,‘Segoe UI’,sans-serif”}}>
<div className="fixed inset-0 pointer-events-none z-0">
<div className=“absolute top-0 right-0 w-[600px] h-[600px] rounded-full opacity-[0.025]” style={{background:‘radial-gradient(circle,#1d4ed8,transparent)’,transform:‘translate(30%,-30%)’}}/>
<div className=“absolute bottom-0 left-0 w-[500px] h-[500px] rounded-full opacity-[0.02]” style={{background:‘radial-gradient(circle,#059669,transparent)’,transform:‘translate(-30%,30%)’}}/>
</div>

```
  {/* SIDEBAR */}
  <aside className={`${collapsed?'w-[58px]':'w-[215px]'} relative z-10 flex-shrink-0 flex flex-col transition-all duration-300 border-l border-slate-800/40`} style={{background:'rgba(5,9,20,0.98)'}}>
    <div className={`flex items-center ${collapsed?'justify-center':'gap-3 px-4'} py-4 border-b border-slate-800/40`}>
      <div className="w-8 h-8 rounded-xl flex items-center justify-center font-black text-white text-sm flex-shrink-0" style={{background:'linear-gradient(135deg,#1d4ed8,#059669)'}}>{APP_LOGO}</div>
      {!collapsed&&<div className="flex-1 min-w-0"><h1 className="text-sm font-black text-white leading-none">{APP_NAME}</h1><p className="text-xs text-slate-600 mt-0.5 truncate">{user.name}</p></div>}
    </div>
    <nav className="flex-1 py-2 px-1.5 space-y-0.5 overflow-y-auto">
      {navItems.map((item:any)=>(
        <button key={item.id} onClick={()=>setTab(item.id)} title={collapsed?item.label:undefined}
          className={`w-full flex items-center ${collapsed?'justify-center px-0':'gap-2.5 px-3'} py-2.5 rounded-xl transition-all duration-200 relative ${tab===item.id?'text-white':'text-slate-500 hover:text-slate-300 hover:bg-slate-800/40'}`}
          style={tab===item.id?{background:'linear-gradient(135deg,rgba(29,78,216,0.2),rgba(5,150,105,0.15))',boxShadow:'inset 0 0 0 1px rgba(29,78,216,0.25)'}:{}}>
          {tab===item.id&&<div className="absolute left-0 top-1/2 -translate-y-1/2 w-0.5 h-5 rounded-full" style={{background:'linear-gradient(to bottom,#1d4ed8,#059669)'}}/>}
          <item.icon size={16} className="flex-shrink-0"/>
          {!collapsed&&<span className="font-semibold text-[13px] flex-1 text-right">{item.label}</span>}
          {!collapsed&&item.badge&&<span className={`text-xs px-1.5 py-0.5 rounded-full font-bold text-white ${item.red?'bg-rose-500':'bg-slate-700'}`}>{item.badge}</span>}
          {collapsed&&item.badge&&<div className={`absolute top-1.5 left-1.5 w-1.5 h-1.5 rounded-full ${item.red?'bg-rose-500':'bg-slate-600'}`}/>}
        </button>
      ))}
    </nav>
    <div className="p-1.5 border-t border-slate-800/40 space-y-0.5">
      {!collapsed&&<div className="px-3 py-2"><Badge text={user.role==='admin'?'مدير عام':user.role==='manager'?'مدير':'موظف'} color={user.role==='admin'?'bg-blue-500/15 text-blue-400':user.role==='manager'?'bg-emerald-500/15 text-emerald-400':'bg-slate-700 text-slate-400'}/><p className="text-xs text-slate-700 mt-1 truncate">{user.branch}</p></div>}
      <button onClick={()=>setCollapsed(!collapsed)} className={`w-full flex items-center ${collapsed?'justify-center':'gap-2.5 px-3'} py-2 rounded-xl text-slate-600 hover:text-slate-400 hover:bg-slate-800/40 transition-all`}><Layers size={15}/>{!collapsed&&<span className="text-xs font-semibold">طي</span>}</button>
      <button onClick={()=>setUser(null)} className={`w-full flex items-center ${collapsed?'justify-center':'gap-2.5 px-3'} py-2 rounded-xl text-rose-600/50 hover:text-rose-400 hover:bg-rose-500/10 transition-all`}><LogOut size={15}/>{!collapsed&&<span className="text-xs font-semibold">خروج</span>}</button>
    </div>
  </aside>

  {/* MAIN */}
  <main className="flex-1 flex flex-col overflow-hidden relative z-10">
    <header className="flex items-center gap-3 px-6 py-3 border-b border-slate-800/40 flex-shrink-0" style={{background:'rgba(5,9,20,0.95)'}}>
      <div className="flex items-center gap-2 text-sm"><span className="text-slate-700">{APP_NAME}</span><ChevronRight size={12} className="rotate-180 text-slate-700"/><span className="text-slate-300 font-semibold">{titles[tab]}</span></div>
      <div className="flex-1"/>
      <div className="flex items-center gap-2">
        <div className="flex items-center gap-2 px-3 py-1.5 rounded-xl border border-slate-800 bg-slate-900/60"><Search size={12} className="text-slate-600"/><input value={search} onChange={e=>setSearch(e.target.value)} placeholder="بحث..." className="bg-transparent outline-none text-slate-300 placeholder-slate-700 w-20 text-right text-xs"/></div>
        <div className="flex items-center gap-1.5 px-2.5 py-1.5 rounded-xl border border-slate-800 bg-slate-900/60 text-xs text-slate-500"><Clock size={11}/>{time.toLocaleTimeString('ar-SA',{hour:'2-digit',minute:'2-digit',second:'2-digit'})}</div>
        <button onClick={()=>setChatOpen(!chatOpen)} className={`p-2 rounded-xl transition-colors relative ${chatOpen?'text-white':'hover:bg-slate-800/60 text-slate-400'}`} style={chatOpen?{background:'linear-gradient(135deg,#1d4ed8,#059669)'}:{}}><Brain size={16}/></button>
        <button onClick={()=>setTab('notifications')} className="relative p-2 rounded-xl hover:bg-slate-800/60 transition-colors"><Bell size={16} className="text-slate-400"/>{unread>0&&<div className="absolute top-1 right-1 w-1.5 h-1.5 bg-rose-500 rounded-full"/>}</button>
        <div className="w-8 h-8 rounded-xl flex items-center justify-center font-black text-white text-xs" style={{background:'linear-gradient(135deg,#1d4ed8,#059669)'}}>{user.avatar}</div>
      </div>
    </header>

    <div className="flex-1 flex overflow-hidden">
      <div className="flex-1 overflow-y-auto p-6">

        {/* ══ DASHBOARD ══ */}
        {tab==='dashboard'&&(
          <div className="space-y-5">
            <div className="flex items-center justify-between">
              <div><h2 className="text-xl font-black text-white">مرحباً، {user.name} 👋</h2><p className="text-slate-500 text-xs mt-0.5">{new Date().toLocaleDateString('ar-SA',{weekday:'long',year:'numeric',month:'long',day:'numeric'})}</p></div>
              <button onClick={printReport} className="flex items-center gap-2 text-white px-3.5 py-2 rounded-xl font-bold text-sm hover:opacity-90" style={{background:'linear-gradient(135deg,#1d4ed8,#059669)'}}><Printer size={14}/>تقرير</button>
            </div>
            <div className="grid grid-cols-2 lg:grid-cols-4 gap-4">
              <KpiCard label="المبيعات" value={fmt(totalSales)} sub={`قبل الضريبة: ${fmt(totalSalesSubtotal)}`} icon={TrendingUp} grad="from-blue-600 to-blue-400" trend="+٨٪" trendUp onClick={()=>setTab('sales')}/>
              <KpiCard label="المشتريات" value={fmt(totalPurchases)} sub={`${purchases.length} عملية`} icon={ArrowDownLeft} grad="from-emerald-600 to-emerald-400" onClick={()=>setTab('purchases')}/>
              <KpiCard label="صافي الربح" value={fmt(netProfit)} sub={`هامش ${pct(netProfit,totalSales)}٪`} icon={DollarSign} grad="from-violet-600 to-violet-400" onClick={()=>setTab('analytics')}/>
              <KpiCard label="ضريبة القيمة المضافة" value={fmt(totalSalesVat)} sub="إجمالي محصل" icon={Percent} grad="from-amber-600 to-amber-400"/>
            </div>
            <div className="grid grid-cols-1 lg:grid-cols-3 gap-5">
              <GCard className="lg:col-span-2 p-5">
                <h3 className="font-black text-white mb-4 flex items-center gap-2"><BarChart3 size={16} className="text-blue-400"/>أداء الفروع</h3>
                <div className="space-y-3.5">
                  {[...activeSales].sort((a,b)=>b.total-a.total).map((s,i)=>(
                    <div key={s.id}><div className="flex justify-between items-center mb-1.5">
                      <div className="flex items-center gap-2"><span className="text-sm font-semibold text-slate-200">{s.branch}</span>{i===0&&<Badge text="الأعلى" color="bg-amber-500/15 text-amber-400"/>}</div>
                      <div className="text-right"><p className="text-sm font-black text-white">{fmt(s.total)}</p><p className="text-xs text-amber-500/80">ض: {fmt(s.vatAmount)}</p></div>
                    </div><BarUI v={pct(s.total,totalSales)} grad={grads[i%grads.length]}/></div>
                  ))}
                </div>
              </GCard>
              <div className="space-y-4">
                <GCard className="p-5">
                  <h3 className="font-black text-white mb-3 flex items-center gap-2 text-sm"><Activity size={14} className="text-emerald-400"/>ملخص مالي</h3>
                  <div className="space-y-2">
                    <Chip label="مبيعات (قبل ضريبة)" value={fmt(totalSalesSubtotal)} vc="text-blue-400"/>
                    <Chip label="المشتريات" value={fmt(totalPurchases)} vc="text-emerald-400"/>
                    <Chip label="المصروفات" value={fmt(totalExpenses)} vc="text-rose-400"/>
                    <Chip label="ضريبة القيمة المضافة" value={fmt(totalSalesVat)} vc="text-amber-400"/>
                    <Chip label="صافي الربح" value={fmt(netProfit)} vc={netProfit>=0?'text-violet-400':'text-rose-400'}/>
                  </div>
                </GCard>
                <GCard className="p-4">
                  <div className="flex justify-between items-center mb-2">
                    <h3 className="font-black text-white text-sm flex items-center gap-2"><Sparkles size={13} className="text-blue-400"/>تحليل AI</h3>
                    <button onClick={getAiInsight} disabled={aiLoading} className="text-xs px-2.5 py-1.5 rounded-lg font-bold text-white disabled:opacity-50" style={{background:'linear-gradient(135deg,#1d4ed8,#059669)'}}>{aiLoading?<RefreshCw size={10} className="animate-spin inline"/>:<><Zap size={10} className="inline ml-1"/>تحليل</>}</button>
                  </div>
                  {aiInsight?<p className="text-slate-300 text-xs leading-relaxed whitespace-pre-wrap">{aiInsight}</p>:<p className="text-slate-600 text-xs">اضغط لتحليل بياناتك بالذكاء الاصطناعي.</p>}
                </GCard>
              </div>
            </div>
            {lowStock.length>0&&<GCard className="p-4" style={{borderColor:'rgba(239,68,68,0.2)'}}>
              <h3 className="font-black text-rose-400 mb-3 flex items-center gap-2 text-sm"><AlertCircle size={14}/>مخزون منخفض ({lowStock.length})</h3>
              <div className="grid grid-cols-2 lg:grid-cols-4 gap-2">
                {lowStock.map(p=><div key={p.id} onClick={()=>setTab('inventory')} className="bg-rose-500/8 border border-rose-500/15 rounded-xl p-3 cursor-pointer hover:bg-rose-500/12 transition-colors"><p className="font-bold text-white text-sm">{p.name}</p><p className="text-rose-400 text-xs">{p.stock}/{p.minStock} {p.unit}</p></div>)}
              </div>
            </GCard>}
          </div>
        )}

        {/* ══ POS ══ */}
        {tab==='pos'&&(
          <div className="space-y-4">
            <div className="flex items-center justify-between">
              <div><h3 className="text-xl font-black text-white flex items-center gap-2"><Monitor size={19} className="text-blue-400"/>نقطة البيع</h3></div>
              {posMsg&&<div className="flex items-center gap-2 bg-emerald-500/15 border border-emerald-500/30 text-emerald-400 font-bold text-sm px-4 py-2 rounded-xl"><CheckCircle2 size={15}/>{posMsg}</div>}
            </div>
            <div className="grid grid-cols-1 lg:grid-cols-3 gap-5">
              <div className="lg:col-span-2 space-y-3">
                <div className="flex items-center gap-2 px-4 py-2.5 rounded-xl border border-slate-700/50 bg-slate-900/60"><Search size={14} className="text-slate-500"/><input value={posSearch} onChange={e=>setPosSearch(e.target.value)} placeholder="ابحث عن منتج..." className="bg-transparent outline-none text-slate-300 placeholder-slate-600 flex-1 text-right text-sm"/></div>
                <div className="grid grid-cols-2 lg:grid-cols-3 gap-3">
                  {products.filter(p=>p.name.includes(posSearch)||p.barcode.includes(posSearch)).map(p=>(
                    <GCard key={p.id} glow="from-blue-500 to-emerald-500" onClick={()=>addToCart(p)} className={`p-4 ${p.stock===0?'opacity-40 cursor-not-allowed':''}`}>
                      <div className="flex justify-between items-start mb-2">
                        <div className={`p-2 rounded-xl bg-gradient-to-br ${grads[p.id%grads.length]} opacity-80`}><Package size={16} className="text-white"/></div>
                        <Badge text={`${p.stock} ${p.unit}`} color={p.stock<=p.minStock?'bg-rose-500/15 text-rose-400':'bg-emerald-500/15 text-emerald-400'}/>
                      </div>
                      <p className="font-black text-white text-sm mb-0.5">{p.name}</p>
                      <p className="text-xs text-slate-500 mb-2" style={{display:'-webkit-box',WebkitLineClamp:2,WebkitBoxOrient:'vertical',overflow:'hidden'}}>{p.description}</p>
                      <div className="border-t border-slate-700/40 pt-2 space-y-1">
                        <div className="flex justify-between"><span className="text-xs text-slate-600">قبل الضريبة</span><span className="text-xs text-slate-400">{fmt(p.unitPrice)}</span></div>
                        <div className="flex justify-between"><span className="text-xs text-amber-600/80">ضريبة {p.vatRate}٪</span><span className="text-xs text-amber-500">+{fmt(calcVat(p.unitPrice,p.vatRate))}</span></div>
                        <div className="flex justify-between"><span className="text-xs font-bold text-slate-300">الإجمالي</span><span className="font-black text-blue-400 text-sm">{fmt(p.priceAfterVat)}</span></div>
                      </div>
                    </GCard>
                  ))}
                </div>
              </div>
              <GCard className="p-5 flex flex-col h-fit sticky top-0">
                <h4 className="font-black text-white mb-4 flex items-center gap-2"><ShoppingBag size={16} className="text-blue-400"/>السلة ({posCart.length})</h4>
                {posCart.length===0?<p className="text-slate-600 text-sm text-center py-8">لا توجد منتجات</p>:(
                  <div className="space-y-2 mb-4">
                    {posCart.map(item=>(
                      <div key={item.productId} className="p-2.5 bg-slate-800/40 rounded-xl">
                        <div className="flex items-center gap-2 mb-1.5"><span className="font-semibold text-white text-xs flex-1">{item.name}</span><button onClick={()=>removeFromCart(item.productId)} className="text-slate-600 hover:text-rose-400"><X size={13}/></button></div>
                        <div className="flex items-center justify-between">
                          <div className="flex items-center gap-1">
                            <button onClick={()=>updateQty(item.productId,item.qty-1)} className="w-6 h-6 rounded-lg bg-slate-700 hover:bg-slate-600 flex items-center justify-center"><Minus size={10} className="text-white"/></button>
                            <span className="w-6 text-center text-white text-xs font-bold">{item.qty}</span>
                            <button onClick={()=>updateQty(item.productId,item.qty+1)} className="w-6 h-6 rounded-lg bg-slate-700 hover:bg-slate-600 flex items-center justify-center"><Plus size={10} className="text-white"/></button>
                          </div>
                          <div className="text-right"><p className="text-xs text-blue-400 font-black">{fmt(item.priceAfterVat*item.qty)}</p><p className="text-xs text-amber-600/70">ض:{fmt(calcVat(item.unitPrice,item.vatRate)*item.qty)}</p></div>
                        </div>
                      </div>
                    ))}
                  </div>
                )}
                {posCart.length>0&&(
                  <div className="space-y-3 border-t border-slate-700/50 pt-4">
                    <div className="flex items-center gap-2"><span className="text-xs text-slate-400 font-semibold whitespace-nowrap">خصم %</span><input type="number" value={posDiscount} onChange={e=>setPosDiscount(Math.min(100,Math.max(0,parseFloat(e.target.value)||0)))} className="flex-1 bg-slate-800/70 border border-slate-700/50 rounded-lg px-3 py-1.5 text-white text-sm outline-none text-center" min="0" max="100"/></div>
                    <div className="rounded-xl bg-slate-800/40 border border-slate-700/30 p-3 space-y-1.5">
                      <div className="flex justify-between text-xs"><span className="text-slate-500">قبل الضريبة</span><span className="text-slate-300">{fmt(posSubtotal)}</span></div>
                      <div className="flex justify-between text-xs"><span className="text-amber-500/80">الضريبة</span><span className="text-amber-400">+{fmt(posVatAmt)}</span></div>
                      {posDiscount>0&&<div className="flex justify-between text-xs"><span className="text-emerald-500/80">خصم {posDiscount}٪</span><span className="text-emerald-400">-{fmt(posDiscountAmt)}</span></div>}
                      <div className="flex justify-between font-black border-t border-slate-700/40 pt-1.5"><span className="text-white">الإجمالي</span><span className="text-blue-400 text-base">{fmt(posFinal)}</span></div>
                    </div>
                    <button onClick={checkout} className="w-full text-white py-3 rounded-xl font-black text-sm hover:opacity-90" style={{background:'linear-gradient(135deg,#1d4ed8,#059669)'}}><Calculator size={15} className="inline ml-2"/>إتمام البيع</button>
                    <button onClick={()=>{setPosCart([]);setPosDiscount(0);}} className="w-full py-2 rounded-xl text-xs text-slate-600 hover:text-rose-400 hover:bg-rose-500/8 transition-all border border-slate-700/30">مسح السلة</button>
                  </div>
                )}
              </GCard>
            </div>
          </div>
        )}

        {/* ══ SALES ══ */}
        {tab==='sales'&&(
          <div className="space-y-4">
            <div className="flex justify-between items-center">
              <div>
                <h3 className="text-xl font-black text-white">المبيعات</h3>
                <div className="flex gap-4 mt-1">
                  <span className="text-xs text-slate-400">قبل الضريبة: <span className="text-blue-400 font-bold">{fmt(totalSalesSubtotal)}</span></span>
                  <span className="text-xs text-slate-400">الضريبة: <span className="text-amber-400 font-bold">{fmt(totalSalesVat)}</span></span>
                  <span className="text-xs text-slate-400">الإجمالي: <span className="text-white font-black">{fmt(totalSales)}</span></span>
                </div>
              </div>
            </div>
            <SubTabs tabs={[['branches','🏪 مبيعات الفروع'],['items','📦 المنتجات المباعة']]} active={salesSubTab} onChange={v=>setSalesSubTab(v as any)}/>

            {salesSubTab==='branches'&&(
              <div className="space-y-3">
                <div className="flex justify-end">
                  <button onClick={()=>setSales(p=>[{id:Date.now(),branch:'فرع جديد',date:today(),network:0,cash:0,subtotal:0,vatAmount:0,total:0,status:'completed'},...p])} className="flex items-center gap-2 text-white px-4 py-2 rounded-xl font-bold text-sm hover:opacity-90" style={{background:'linear-gradient(135deg,#1d4ed8,#2563eb)'}}><Plus size={14}/>إضافة فرع</button>
                </div>
                <GCard className="overflow-hidden">
                  <table className="w-full text-sm">
                    <thead><tr className="border-b border-slate-800/50" style={{background:'rgba(12,20,40,0.9)'}}>
                      {['الفرع','التاريخ','شبكة','نقدي','قبل الضريبة','الضريبة','الإجمالي','الحالة','✕'].map(h=><th key={h} className="p-3 text-right text-xs font-bold text-slate-500">{h}</th>)}
                    </tr></thead>
                    <tbody>
                      {sales.map(s=>(
                        <tr key={s.id} className="hover:bg-slate-800/15 transition-colors">
                          <EditCell value={s.branch} onChange={v=>setSales(p=>p.map(x=>x.id===s.id?{...x,branch:v}:x))} className="font-semibold text-slate-200"/>
                          <EditCell value={s.date} onChange={v=>setSales(p=>p.map(x=>x.id===s.id?{...x,date:v}:x))} className="text-slate-500 text-xs"/>
                          <EditCell value={String(s.network)} onChange={v=>{const n=parseFloat(v)||0;setSales(p=>p.map(x=>{if(x.id!==s.id)return x;const t=n+x.cash,sub=parseFloat((t/1.15).toFixed(2));return{...x,network:n,total:t,subtotal:sub,vatAmount:parseFloat((t-sub).toFixed(2))}}));}} className="text-violet-400 font-semibold"/>
                          <EditCell value={String(s.cash)} onChange={v=>{const c=parseFloat(v)||0;setSales(p=>p.map(x=>{if(x.id!==s.id)return x;const t=x.network+c,sub=parseFloat((t/1.15).toFixed(2));return{...x,cash:c,total:t,subtotal:sub,vatAmount:parseFloat((t-sub).toFixed(2))}}));}} className="text-emerald-400 font-semibold"/>
                          <td className="border-b border-slate-800/40 p-3 text-slate-400 text-xs">{fmt(s.subtotal)}</td>
                          <td className="border-b border-slate-800/40 p-3 text-amber-400 text-xs">{fmt(s.vatAmount)}</td>
                          <td className="border-b border-slate-800/40 p-3 font-black text-blue-400">{fmt(s.total)}</td>
                          <td className="border-b border-slate-800/40 p-3">
                            <select value={s.status} onChange={e=>setSales(p=>p.map(x=>x.id===s.id?{...x,status:e.target.value as any}:x))} className="bg-slate-800/70 border border-slate-700/40 rounded-lg px-2 py-1 text-xs text-white outline-none">
                              <option value="completed">مكتمل</option><option value="pending">معلق</option><option value="cancelled">ملغي</option>
                            </select>
                          </td>
                          <td className="border-b border-slate-800/40 p-3"><button onClick={()=>setSales(sales.filter(x=>x.id!==s.id))} className="text-slate-700 hover:text-rose-400 transition-colors"><Trash2 size={14}/></button></td>
                        </tr>
                      ))}
                    </tbody>
                    <tfoot><tr className="border-t border-slate-800" style={{background:'rgba(12,20,40,0.7)'}}>
                      <td colSpan={4} className="p-3 text-slate-500 font-bold text-xs">الإجمالي</td>
                      <td className="p-3 font-bold text-slate-400">{fmt(totalSalesSubtotal)}</td>
                      <td className="p-3 font-bold text-amber-400">{fmt(totalSalesVat)}</td>
                      <td className="p-3 font-black text-blue-400 text-base">{fmt(totalSales)}</td>
                      <td colSpan={2}/>
                    </tr></tfoot>
                  </table>
                </GCard>
              </div>
            )}

            {salesSubTab==='items'&&(
              <div className="space-y-4">
                <GCard className="p-5">
                  <h4 className="font-black text-white mb-4 flex items-center gap-2 text-sm"><Plus size={15} className="text-emerald-400"/>تسجيل منتج مباع</h4>
                  <div className="grid grid-cols-2 lg:grid-cols-4 gap-3">
                    <Field label="المنتج" req>
                      <Sel value={soldItemForm.productId} onChange={v=>setSoldItemForm(f=>({...f,productId:v}))}>
                        <option value="">-- اختر منتج --</option>
                        {products.map(p=><option key={p.id} value={p.id}>{p.name} ({p.stock} {p.unit})</option>)}
                      </Sel>
                    </Field>
                    <Field label="الفرع" req>
                      <Sel value={soldItemForm.branch} onChange={v=>setSoldItemForm(f=>({...f,branch:v}))}>
                        <option value="">-- اختر فرع --</option>
                        {branchList.map(b=><option key={b} value={b}>{b}</option>)}
                        <option value="__new__">+ فرع جديد</option>
                      </Sel>
                    </Field>
                    {soldItemForm.branch==='__new__'&&(
                      <Field label="اسم الفرع الجديد">
                        <Inp value={soldItemForm.customBranch} onChange={v=>setSoldItemForm(f=>({...f,customBranch:v}))}/>
                      </Field>
                    )}
                    <Field label="الكمية" req>
                      <Inp value={soldItemForm.qty} onChange={v=>setSoldItemForm(f=>({...f,qty:v}))} type="number" ph="1"/>
                    </Field>
                    <Field label="التاريخ">
                      <Inp value={soldItemForm.date} onChange={v=>setSoldItemForm(f=>({...f,date:v}))} type="date"/>
                    </Field>
                  </div>
                  {(()=>{
                    const p=products.find(x=>x.id===parseInt(soldItemForm.productId));
                    const q=parseFloat(soldItemForm.qty)||0;
                    if(!p||q<=0)return null;
                    const notEnough=q>p.stock;
                    return(
                      <div className={`mt-3 rounded-xl p-3 flex items-center justify-between border ${notEnough?'bg-rose-500/8 border-rose-500/20':'bg-emerald-500/5 border-emerald-500/15'}`}>
                        <div className="flex items-center gap-3">
                          <span className="text-sm text-slate-300">{p.name} × {q} {p.unit}</span>
                          <Badge text={notEnough?`المخزون ${p.stock} فقط!`:`متوفر: ${p.stock}`} color={notEnough?'bg-rose-500/15 text-rose-400':'bg-emerald-500/15 text-emerald-400'}/>
                        </div>
                        {!notEnough&&<div className="text-right">
                          <p className="text-sm font-black text-emerald-400">{fmt(p.priceAfterVat*q)}</p>
                          <p className="text-xs text-amber-500/70">ضريبة: {fmt(calcVat(p.unitPrice,p.vatRate)*q)}</p>
                        </div>}
                      </div>
                    );
                  })()}
                  <div className="mt-3 flex justify-end">
                    <button onClick={addSoldItem}
                      disabled={!soldItemForm.productId||!soldItemForm.qty||!soldItemForm.branch||(soldItemForm.branch==='__new__'&&!soldItemForm.customBranch)}
                      className="flex items-center gap-2 text-white px-5 py-2.5 rounded-xl font-bold text-sm hover:opacity-90 disabled:opacity-40"
                      style={{background:'linear-gradient(135deg,#059669,#047857)'}}><CheckCircle2 size={14}/>تسجيل البيع</button>
                  </div>
                </GCard>
                {soldItems.length>0?(
                  <GCard className="overflow-hidden">
                    <div className="p-4 border-b border-slate-800/50 flex justify-between items-center">
                      <h4 className="font-black text-white text-sm">المنتجات المباعة ({soldItems.length})</h4>
                      <div className="flex gap-4 text-xs">
                        <span className="text-amber-400">الضريبة: {fmt(soldItems.reduce((a,i)=>a+(calcVat(i.unitPrice,i.vatRate)*i.qty),0))}</span>
                        <span className="text-emerald-400 font-black">الإجمالي: {fmt(soldItems.reduce((a,i)=>a+(i.priceAfterVat*i.qty),0))}</span>
                      </div>
                    </div>
                    <table className="w-full text-sm">
                      <thead><tr className="border-b border-slate-800/50" style={{background:'rgba(12,20,40,0.9)'}}>
                        {['المنتج','الفرع','التاريخ','الكمية','سعر الوحدة','الإجمالي','✕'].map(h=><th key={h} className="p-3 text-right text-xs font-bold text-slate-500">{h}</th>)}
                      </tr></thead>
                      <tbody>
                        {soldItems.map(item=>(
                          <tr key={item.id} className="hover:bg-slate-800/15 transition-colors">
                            <td className="border-b border-slate-800/40 p-3"><p className="font-bold text-white text-sm">{item.productName}</p><p className="text-xs text-slate-600">{item.barcode}</p></td>
                            <td className="border-b border-slate-800/40 p-3"><Badge text={item.branch} color="bg-slate-800 text-slate-300"/></td>
                            <td className="border-b border-slate-800/40 p-3 text-slate-500 text-xs">{item.date}</td>
                            <td className="border-b border-slate-800/40 p-3 font-black text-white">{item.qty} <span className="text-xs text-slate-500">{item.unit}</span></td>
                            <td className="border-b border-slate-800/40 p-3"><p className="text-slate-300 text-xs">{fmt(item.unitPrice)}</p><p className="text-amber-500/70 text-xs">+{fmt(calcVat(item.unitPrice,item.vatRate))}</p></td>
                            <td className="border-b border-slate-800/40 p-3 font-black text-emerald-400">{fmt(item.priceAfterVat*item.qty)}</td>
                            <td className="border-b border-slate-800/40 p-3"><button onClick={()=>{setProducts(p=>p.map(x=>x.id===item.productId?{...x,stock:x.stock+item.qty}:x));setSoldItems(s=>s.filter(x=>x.id!==item.id));}} className="text-slate-700 hover:text-rose-400 transition-colors"><Trash2 size={14}/></button></td>
                          </tr>
                        ))}
                      </tbody>
                      <tfoot><tr className="border-t border-slate-800" style={{background:'rgba(12,20,40,0.7)'}}>
                        <td colSpan={5} className="p-3 text-slate-500 font-bold text-xs">الإجمالي</td>
                        <td className="p-3 font-black text-emerald-400 text-base">{fmt(soldItems.reduce((a,i)=>a+(i.priceAfterVat*i.qty),0))}</td>
                        <td/>
                      </tr></tfoot>
                    </table>
                  </GCard>
                ):<EmptyState icon={Package} msg="لا توجد مبيعات مسجلة" sub="اختر منتج وفرع لتسجيل المبيعات"/>}
              </div>
            )}
          </div>
        )}

        {/* ══ PURCHASES ══ */}
        {tab==='purchases'&&(
          <div className="space-y-4">
            <div className="flex justify-between items-center">
              <div>
                <h3 className="text-xl font-black text-white flex items-center gap-2"><ArrowDownLeft size={19} className="text-emerald-400"/>المشتريات</h3>
                <div className="flex gap-4 mt-1">
                  <span className="text-xs text-slate-400">إجمالي المستلم: <span className="text-emerald-400 font-bold">{fmt(totalPurchases)}</span></span>
                  <span className="text-xs text-slate-400">{purchases.length} عملية</span>
                </div>
              </div>
            </div>
            <SubTabs tabs={[['list','📋 سجل المشتريات'],['add','➕ شراء جديد']]} active={purchSubTab} onChange={v=>setPurchSubTab(v as any)}/>

            {purchSubTab==='add'&&(
              <GCard className="p-6">
                <h4 className="font-black text-white mb-5 flex items-center gap-2"><Truck size={16} className="text-emerald-400"/>تسجيل مشتريات من مورد</h4>
                <div className="grid grid-cols-1 lg:grid-cols-2 gap-4 mb-4">
                  <Field label="المورد" req>
                    <Sel value={purchForm.supplierId} onChange={v=>setPurchForm(f=>({...f,supplierId:v}))}>
                      <option value="">-- اختر مورد --</option>
                      {suppliers.map(s=><option key={s.id} value={s.id}>{s.name}</option>)}
                    </Sel>
                  </Field>
                  <Field label="التاريخ">
                    <Inp value={purchForm.date} onChange={v=>setPurchForm(f=>({...f,date:v}))} type="date"/>
                  </Field>
                </div>
                <div className="grid grid-cols-1 lg:grid-cols-3 gap-4 mb-4">
                  <Field label="المنتج" req>
                    <Sel value={purchForm.productId} onChange={v=>setPurchForm(f=>({...f,productId:v,customUnitPrice:''}))}>
                      <option value="">-- اختر منتج --</option>
                      {products.map(p=><option key={p.id} value={p.id}>{p.name} (مخزون: {p.stock})</option>)}
                    </Sel>
                  </Field>
                  <Field label="الكمية" req>
                    <Inp value={purchForm.qty} onChange={v=>setPurchForm(f=>({...f,qty:v}))} type="number" ph="1"/>
                  </Field>
                  <Field label="سعر الشراء (اختياري)">
                    <Inp value={purchForm.customUnitPrice} onChange={v=>setPurchForm(f=>({...f,customUnitPrice:v}))} type="number" ph="سعر التكلفة قبل ضريبة"/>
                  </Field>
                </div>
                <Field label="ملاحظات">
                  <Inp value={purchForm.notes} onChange={v=>setPurchForm(f=>({...f,notes:v}))} ph="ملاحظات اختيارية..."/>
                </Field>
                {(()=>{
                  const p=products.find(x=>x.id===parseInt(purchForm.productId));
                  const q=parseFloat(purchForm.qty)||0;
                  if(!p||q<=0)return null;
                  const unitPrice=purchForm.customUnitPrice?parseFloat(purchForm.customUnitPrice):p.unitPrice;
                  const priceAV=calcAfterVat(unitPrice,p.vatRate);
                  return(
                    <div className="mt-4 rounded-2xl border border-emerald-500/20 bg-emerald-500/4 p-4">
                      <p className="text-xs font-bold text-emerald-400 uppercase mb-3 tracking-wider">معاينة الشراء</p>
                      <div className="grid grid-cols-2 lg:grid-cols-4 gap-3 mb-3">
                        <Chip label="سعر الوحدة" value={fmt(unitPrice)} vc="text-slate-300"/>
                        <Chip label={`ضريبة ${p.vatRate}٪`} value={`+${fmt(calcVat(unitPrice,p.vatRate))}`} vc="text-amber-400"/>
                        <Chip label="الوحدة بعد الضريبة" value={fmt(priceAV)} vc="text-emerald-400"/>
                        <Chip label={`إجمالي (${q} ${p.unit})`} value={fmt(priceAV*q)} vc="text-blue-400"/>
                      </div>
                      <div className="flex items-center gap-3 text-xs text-slate-400 bg-slate-800/40 rounded-xl p-2.5">
                        <Package size={12} className="text-emerald-400"/>
                        <span>المخزون الحالي: <span className="font-bold text-white">{p.stock} {p.unit}</span></span>
                        <span className="text-slate-600">→ بعد الاستلام:</span>
                        <span className="font-bold text-emerald-400">{p.stock+q} {p.unit}</span>
                      </div>
                    </div>
                  );
                })()}
                <div className="mt-5 flex gap-3">
                  <button onClick={addPurchase} disabled={!purchForm.supplierId||!purchForm.productId||!purchForm.qty}
                    className="flex items-center gap-2 text-white px-6 py-2.5 rounded-xl font-bold text-sm hover:opacity-90 disabled:opacity-40"
                    style={{background:'linear-gradient(135deg,#059669,#047857)'}}><CheckCircle2 size={15}/>تأكيد الشراء وتحديث المخزون</button>
                  <button onClick={()=>setPurchSubTab('list')} className="px-5 py-2.5 rounded-xl text-sm font-bold text-slate-400 hover:bg-slate-800 border border-slate-700/40 transition-colors">إلغاء</button>
                </div>
              </GCard>
            )}

            {purchSubTab==='list'&&(
              <>
                <div className="flex justify-end">
                  <button onClick={()=>setPurchSubTab('add')} className="flex items-center gap-2 text-white px-4 py-2 rounded-xl font-bold text-sm hover:opacity-90" style={{background:'linear-gradient(135deg,#059669,#047857)'}}><Plus size={14}/>شراء جديد</button>
                </div>
                {purchases.length>0?(
                  <GCard className="overflow-hidden">
                    <table className="w-full text-sm">
                      <thead><tr className="border-b border-slate-800/50" style={{background:'rgba(12,20,40,0.9)'}}>
                        {['المنتج','المورد','التاريخ','الكمية','سعر الوحدة','ضريبة%','الإجمالي','الحالة','✕'].map(h=><th key={h} className="p-3 text-right text-xs font-bold text-slate-500">{h}</th>)}
                      </tr></thead>
                      <tbody>
                        {purchases.map(p=>(
                          <tr key={p.id} className="hover:bg-slate-800/15 transition-colors">
                            <td className="border-b border-slate-800/40 p-3"><p className="font-bold text-white text-sm">{p.productName}</p>{p.notes&&<p className="text-xs text-slate-600">{p.notes}</p>}</td>
                            <td className="border-b border-slate-800/40 p-3 text-slate-300 text-sm">{p.supplierName}</td>
                            <td className="border-b border-slate-800/40 p-3 text-slate-500 text-xs">{p.date}</td>
                            <td className="border-b border-slate-800/40 p-3 font-black text-white">{p.qty} <span className="text-xs text-slate-500">{p.unit}</span></td>
                            <td className="border-b border-slate-800/40 p-3 text-slate-400 text-xs"><p>{fmt(p.unitPrice)}</p><p className="text-amber-500/70">+{fmt(calcVat(p.unitPrice,p.vatRate))}</p></td>
                            <td className="border-b border-slate-800/40 p-3 text-amber-400 text-xs font-bold">{p.vatRate}٪</td>
                            <td className="border-b border-slate-800/40 p-3 font-black text-emerald-400">{fmt(p.totalCost)}</td>
                            <td className="border-b border-slate-800/40 p-3">
                              <select value={p.status} onChange={e=>setPurchases(prev=>prev.map(x=>x.id===p.id?{...x,status:e.target.value as any}:x))} className="bg-slate-800/70 border border-slate-700/40 rounded-lg px-2 py-1 text-xs text-white outline-none">
                                <option value="received">مستلم</option><option value="pending">معلق</option><option value="cancelled">ملغي</option>
                              </select>
                            </td>
                            <td className="border-b border-slate-800/40 p-3"><button onClick={()=>{if(p.status==='received')setProducts(prev=>prev.map(x=>x.id===p.productId?{...x,stock:Math.max(0,x.stock-p.qty)}:x));setPurchases(prev=>prev.filter(x=>x.id!==p.id));}} className="text-slate-700 hover:text-rose-400 transition-colors"><Trash2 size={14}/></button></td>
                          </tr>
                        ))}
                      </tbody>
                      <tfoot><tr className="border-t border-slate-800" style={{background:'rgba(12,20,40,0.7)'}}>
                        <td colSpan={6} className="p-3 text-slate-500 font-bold text-xs">إجمالي المستلم</td>
                        <td className="p-3 font-black text-emerald-400 text-base">{fmt(totalPurchases)}</td>
                        <td colSpan={2}/>
                      </tr></tfoot>
                    </table>
                  </GCard>
                ):<EmptyState icon={Truck} msg="لا توجد مشتريات مسجلة" sub="اضغط شراء جديد لإضافة عملية شراء" action={()=>setPurchSubTab('add')} actionLabel="شراء جديد"/>}
              </>
            )}
          </div>
        )}

        {/* ══ EXPENSES ══ */}
        {tab==='expenses'&&(
          <div className="space-y-4">
            <div className="flex justify-between items-center">
              <div><h3 className="text-xl font-black text-white">المصروفات</h3><p className="text-slate-500 text-sm">إجمالي: {fmt(totalExpenses)}</p></div>
              <button onClick={()=>setExpenses(p=>[{id:Date.now(),category:'مصروف جديد',amount:0,date:today(),notes:'',recurring:false},...p])} className="flex items-center gap-2 text-white px-4 py-2 rounded-xl font-bold text-sm hover:opacity-90" style={{background:'linear-gradient(135deg,#dc2626,#9f1239)'}}><Plus size={14}/>إضافة</button>
            </div>
            <GCard className="overflow-hidden">
              <table className="w-full text-sm">
                <thead><tr className="border-b border-slate-800/50" style={{background:'rgba(12,20,40,0.9)'}}>
                  {['التصنيف','المبلغ','التاريخ','ملاحظات','نوع','✕'].map(h=><th key={h} className="p-3.5 text-right text-xs font-bold text-slate-500">{h}</th>)}
                </tr></thead>
                <tbody>
                  {expenses.map(e=>(
                    <tr key={e.id} className="hover:bg-slate-800/15 transition-colors">
                      <EditCell value={e.category} onChange={v=>setExpenses(p=>p.map(x=>x.id===e.id?{...x,category:v}:x))} className="font-semibold text-slate-200"/>
                      <EditCell value={String(e.amount)} onChange={v=>setExpenses(p=>p.map(x=>x.id===e.id?{...x,amount:parseFloat(v)||0}:x))} className="font-black text-rose-400"/>
                      <EditCell value={e.date} onChange={v=>setExpenses(p=>p.map(x=>x.id===e.id?{...x,date:v}:x))} className="text-slate-500 text-xs"/>
                      <EditCell value={e.notes} onChange={v=>setExpenses(p=>p.map(x=>x.id===e.id?{...x,notes:v}:x))} className="text-slate-400"/>
                      <td className="border-b border-slate-800/40 p-3.5"><Badge text={e.recurring?'متكرر':'مرة واحدة'} color={e.recurring?'bg-amber-500/15 text-amber-400':'bg-slate-800 text-slate-500'}/></td>
                      <td className="border-b border-slate-800/40 p-3.5"><button onClick={()=>setExpenses(expenses.filter(x=>x.id!==e.id))} className="text-slate-700 hover:text-rose-400 transition-colors"><Trash2 size={14}/></button></td>
                    </tr>
                  ))}
                </tbody>
                <tfoot><tr className="border-t border-slate-800" style={{background:'rgba(12,20,40,0.7)'}}>
                  <td className="p-3 text-slate-500 font-bold text-xs">الإجمالي</td>
                  <td className="p-3 font-black text-rose-400 text-base">{fmt(totalExpenses)}</td>
                  <td colSpan={4}/>
                </tr></tfoot>
              </table>
            </GCard>
          </div>
        )}

        {/* ══ INVENTORY ══ */}
        {tab==='inventory'&&(
          <div className="space-y-4">
            <div className="flex justify-between items-center">
              <div><h3 className="text-xl font-black text-white">المخزون</h3><p className="text-slate-500 text-sm">{products.length} منتج · {lowStock.length} منخفض · قيمة: {fmt(totalInventoryValue)}</p></div>
              <button onClick={()=>openModal('product')} className="flex items-center gap-2 text-white px-4 py-2 rounded-xl font-bold text-sm hover:opacity-90" style={{background:'linear-gradient(135deg,#059669,#047857)'}}><Plus size={14}/>منتج جديد</button>
            </div>
            <div className="grid grid-cols-1 lg:grid-cols-2 gap-4">
              {products.map((p,i)=>{const isLow=p.stock<=p.minStock;return(
                <GCard key={p.id} className="p-5" style={isLow?{borderColor:'rgba(239,68,68,0.25)'}:{}}>
                  <div className="flex justify-between items-start mb-2">
                    <div className="flex-1 ml-3">
                      <div className="flex items-center gap-2 mb-0.5"><h4 className="font-black text-white text-sm">{p.name}</h4>{isLow&&<Badge text="منخفض" color="bg-rose-500/15 text-rose-400"/>}</div>
                      <p className="text-xs text-slate-500 leading-snug mb-1">{p.description}</p>
                      <div className="flex items-center gap-2"><Badge text={p.category} color="bg-slate-700 text-slate-400"/><span className="text-xs text-slate-600">{p.barcode}</span><span className="text-xs text-slate-600">{p.unit}</span></div>
                    </div>
                    <div className="flex gap-1 flex-shrink-0">
                      <button onClick={()=>openModal('product',p)} className="p-1.5 text-slate-600 hover:text-emerald-400 hover:bg-emerald-500/8 rounded-lg transition-colors"><Edit size={13}/></button>
                      <button onClick={()=>setProducts(products.filter(x=>x.id!==p.id))} className="p-1.5 text-slate-600 hover:text-rose-400 hover:bg-rose-500/8 rounded-lg transition-colors"><Trash2 size={13}/></button>
                    </div>
                  </div>
                  <div className="mt-3 rounded-xl bg-slate-800/50 border border-slate-700/30 p-3">
                    <div className="grid grid-cols-3 gap-2 text-center">
                      <div><p className="text-xs text-slate-500 mb-0.5">قبل الضريبة</p><p className="font-black text-slate-300 text-sm">{fmt(p.unitPrice)}</p></div>
                      <div className="border-x border-slate-700/40"><p className="text-xs text-amber-500/80 mb-0.5">ضريبة {p.vatRate}٪</p><p className="font-black text-amber-400 text-sm">+{fmt(calcVat(p.unitPrice,p.vatRate))}</p></div>
                      <div><p className="text-xs text-emerald-500/80 mb-0.5">بعد الضريبة</p><p className="font-black text-emerald-400 text-sm">{fmt(p.priceAfterVat)}</p></div>
                    </div>
                  </div>
                  <div className="flex justify-between text-xs text-slate-500 mt-3 mb-2">
                    <span>المخزون: <span className={`font-bold ${isLow?'text-rose-400':'text-emerald-400'}`}>{p.stock} {p.unit}</span></span>
                    <span>الحد: <span className="font-bold text-slate-400">{p.minStock}</span></span>
                    <span>القيمة: <span className="font-bold text-blue-400">{fmt(p.priceAfterVat*p.stock)}</span></span>
                  </div>
                  <BarUI v={pct(p.stock,p.minStock*3)} grad={isLow?'from-rose-500 to-rose-400':'from-emerald-500 to-teal-400'}/>
                </GCard>
              );})}
            </div>
          </div>
        )}

        {/* ══ GOALS ══ */}
        {tab==='goals'&&(
          <div className="space-y-4">
            <div className="flex justify-between items-center">
              <div><h3 className="text-xl font-black text-white">الأهداف</h3><p className="text-slate-500 text-sm">{goals.length} هدف · {goals.filter(g=>pct(g.current,g.target)>=100).length} مكتمل</p></div>
              <button onClick={()=>setGoals(p=>[{id:Date.now(),title:'هدف جديد',target:100000,current:0,deadline:'٣١ ديسمبر',color:'from-blue-500 to-cyan-400',category:'عام'},...p])} className="flex items-center gap-2 text-white px-4 py-2 rounded-xl font-bold text-sm hover:opacity-90" style={{background:'linear-gradient(135deg,#1d4ed8,#059669)'}}><Plus size={14}/>هدف جديد</button>
            </div>
            <div className="grid grid-cols-1 lg:grid-cols-2 gap-4">
              {goals.map(g=>{const p=pct(g.current,g.target);return(
                <GCard key={g.id} className="p-6">
                  <div className="flex justify-between items-start mb-4">
                    <div>
                      <p className="font-black text-white text-base">{g.title}</p>
                      <div className="flex items-center gap-2 mt-1"><Badge text={g.category} color="bg-slate-700 text-slate-400"/><span className="text-xs text-slate-500">{g.deadline}</span></div>
                    </div>
                    <div className="text-left">
                      <div className={`text-2xl font-black bg-gradient-to-r ${g.color} bg-clip-text text-transparent`}>{p}٪</div>
                      {p>=100&&<Badge text="مكتمل ✓" color="bg-emerald-500/15 text-emerald-400"/>}
                    </div>
                  </div>
                  <BarUI v={p} grad={g.color} h="h-3"/>
                  <div className="flex justify-between mt-3 text-xs text-slate-500">
                    <span>المحقق: <span className="font-bold text-white">{g.current.toLocaleString('ar-SA')}</span></span>
                    <span>الهدف: <span className="font-bold text-white">{g.target.toLocaleString('ar-SA')}</span></span>
                    {p<100&&<span className="text-rose-400 font-bold">متبقي: {(g.target-g.current).toLocaleString('ar-SA')}</span>}
                  </div>
                </GCard>
              );})}
            </div>
          </div>
        )}

        {/* ══ CUSTOMERS ══ */}
        {tab==='customers'&&(
          <div className="space-y-5">
            <div className="flex justify-between items-center">
              <h3 className="text-xl font-black text-white">العملاء والموردون</h3>
              <div className="flex gap-2">
                <button onClick={()=>openModal('customer')} className="flex items-center gap-2 text-white px-4 py-2 rounded-xl font-bold text-sm hover:opacity-90" style={{background:'linear-gradient(135deg,#1d4ed8,#2563eb)'}}><Plus size={14}/>عميل</button>
                <button onClick={()=>openModal('supplier')} className="flex items-center gap-2 text-white px-4 py-2 rounded-xl font-bold text-sm hover:opacity-90" style={{background:'linear-gradient(135deg,#059669,#047857)'}}><Plus size={14}/>مورد</button>
              </div>
            </div>
            <div>
              <h4 className="text-sm font-black text-slate-300 mb-3 flex items-center gap-2"><Users size={14} className="text-blue-400"/>العملاء ({customers.length})</h4>
              <div className="grid grid-cols-1 lg:grid-cols-3 gap-4">
                {customers.map(c=>(
                  <GCard key={c.id} className="p-5">
                    <div className="flex justify-between items-start mb-3">
                      <div>
                        <p className="font-black text-white">{c.name}</p>
                        <Badge text={c.type==='wholesale'?'جملة':'تجزئة'} color={c.type==='wholesale'?'bg-blue-500/15 text-blue-400':'bg-slate-700 text-slate-400'}/>
                      </div>
                      <div className="flex gap-1">
                        <button onClick={()=>openModal('customer',c)} className="p-1.5 text-slate-600 hover:text-blue-400 rounded-lg transition-colors"><Edit size={13}/></button>
                        <button onClick={()=>setCustomers(customers.filter(x=>x.id!==c.id))} className="p-1.5 text-slate-600 hover:text-rose-400 rounded-lg transition-colors"><Trash2 size={13}/></button>
                      </div>
                    </div>
                    <div className="space-y-1.5 text-xs text-slate-500">
                      <p className="flex items-center gap-2"><Phone size={11}/>{c.phone}</p>
                      <p className="flex items-center gap-2"><MapPin size={11}/>{c.address}</p>
                    </div>
                    <div className="mt-3 pt-3 border-t border-slate-700/40 grid grid-cols-2 gap-2">
                      <div className="text-center"><p className="text-xs text-slate-500">مستحقات</p><p className="font-black text-rose-400">{fmt(c.balance)}</p></div>
                      <div className="text-center"><p className="text-xs text-slate-500">إجمالي المشتريات</p><p className="font-black text-blue-400">{fmt(c.totalPurchases)}</p></div>
                    </div>
                  </GCard>
                ))}
              </div>
            </div>
            <div>
              <h4 className="text-sm font-black text-slate-300 mb-3 flex items-center gap-2"><Truck size={14} className="text-emerald-400"/>الموردون ({suppliers.length})</h4>
              <div className="grid grid-cols-1 lg:grid-cols-3 gap-4">
                {suppliers.map(s=>(
                  <GCard key={s.id} className="p-5">
                    <div className="flex justify-between items-start mb-3">
                      <div><p className="font-black text-white">{s.name}</p><Badge text={s.category} color="bg-emerald-500/15 text-emerald-400"/></div>
                      <div className="flex gap-1">
                        <button onClick={()=>openModal('supplier',s)} className="p-1.5 text-slate-600 hover:text-emerald-400 rounded-lg transition-colors"><Edit size={13}/></button>
                        <button onClick={()=>setSuppliers(suppliers.filter(x=>x.id!==s.id))} className="p-1.5 text-slate-600 hover:text-rose-400 rounded-lg transition-colors"><Trash2 size={13}/></button>
                      </div>
                    </div>
                    <div className="space-y-1.5 text-xs text-slate-500">
                      <p className="flex items-center gap-2"><Phone size={11}/>{s.phone}</p>
                      <p className="flex items-center gap-2"><MapPin size={11}/>{s.address}</p>
                    </div>
                    <div className="mt-3 pt-3 border-t border-slate-700/40 grid grid-cols-2 gap-2">
                      <div className="text-center"><p className="text-xs text-slate-500">مستحقات</p><p className="font-black text-rose-400">{fmt(s.balance)}</p></div>
                      <div className="text-center"><p className="text-xs text-slate-500">إجمالي الطلبات</p><p className="font-black text-emerald-400">{fmt(s.totalOrders)}</p></div>
                    </div>
                  </GCard>
                ))}
              </div>
            </div>
          </div>
        )}

        {/* ══ INVOICES ══ */}
        {tab==='invoices'&&(
          <div className="space-y-4">
            <div className="flex justify-between items-center">
              <div><h3 className="text-xl font-black text-white">الفواتير</h3>
                <div className="flex gap-4 mt-1">
                  <span className="text-xs text-slate-400">مستحقات: <span className="text-rose-400 font-bold">{fmt(totalReceivables)}</span></span>
                  <span className="text-xs text-slate-400">مدفوعات: <span className="text-amber-400 font-bold">{fmt(totalPayables)}</span></span>
                </div>
              </div>
              <button onClick={()=>openModal('invoice')} className="flex items-center gap-2 text-white px-4 py-2 rounded-xl font-bold text-sm hover:opacity-90" style={{background:'linear-gradient(135deg,#1d4ed8,#059669)'}}><Plus size={14}/>فاتورة جديدة</button>
            </div>
            <GCard className="overflow-hidden">
              <table className="w-full text-sm">
                <thead><tr className="border-b border-slate-800/50" style={{background:'rgba(12,20,40,0.9)'}}>
                  {['رقم','نوع','الطرف','التاريخ','قبل الضريبة','الضريبة','الإجمالي','مدفوع','متبقي','الحالة','تسديد'].map(h=><th key={h} className="p-3 text-right text-xs font-bold text-slate-500">{h}</th>)}
                </tr></thead>
                <tbody>
                  {invoices.map(inv=>(
                    <tr key={inv.id} className="hover:bg-slate-800/15 transition-colors">
                      <td className="border-b border-slate-800/40 p-3 font-black text-blue-400 text-xs">{inv.number}</td>
                      <td className="border-b border-slate-800/40 p-3"><Badge text={inv.type==='sale'?'بيع':'شراء'} color={inv.type==='sale'?'bg-blue-500/15 text-blue-400':'bg-emerald-500/15 text-emerald-400'}/></td>
                      <td className="border-b border-slate-800/40 p-3 font-semibold text-slate-200 text-xs">{inv.party}</td>
                      <td className="border-b border-slate-800/40 p-3 text-slate-500 text-xs">{inv.date}</td>
                      <td className="border-b border-slate-800/40 p-3 text-slate-400 text-xs">{fmt(inv.subtotal)}</td>
                      <td className="border-b border-slate-800/40 p-3 text-amber-400 text-xs">{fmt(inv.vatAmount)}</td>
                      <td className="border-b border-slate-800/40 p-3 font-black text-white">{fmt(inv.total)}</td>
                      <td className="border-b border-slate-800/40 p-3 text-emerald-400 text-xs">{fmt(inv.paid)}</td>
                      <td className="border-b border-slate-800/40 p-3 font-bold text-rose-400 text-xs">{fmt(inv.total-inv.paid)}</td>
                      <td className="border-b border-slate-800/40 p-3"><Badge text={inv.status==='paid'?'مدفوع':inv.status==='partial'?'جزئي':'غير مدفوع'} color={inv.status==='paid'?'bg-emerald-500/15 text-emerald-400':inv.status==='partial'?'bg-amber-500/15 text-amber-400':'bg-rose-500/15 text-rose-400'}/></td>
                      <td className="border-b border-slate-800/40 p-3">{inv.status!=='paid'&&<button onClick={()=>recordPayment(inv.id,inv.total-inv.paid)} className="text-xs px-2 py-1 rounded-lg font-bold text-white hover:opacity-80" style={{background:'linear-gradient(135deg,#059669,#047857)'}}>تسديد</button>}</td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </GCard>
          </div>
        )}

        {/* ══ PAYROLL ══ */}
        {tab==='payroll'&&(
          <div className="space-y-5">
            <div className="flex justify-between items-center">
              <div><h3 className="text-xl font-black text-white">الرواتب والموظفون</h3></div>
              <div className="flex gap-2">
                <button onClick={generatePayroll} className="flex items-center gap-2 text-white px-4 py-2 rounded-xl font-bold text-sm hover:opacity-90" style={{background:'linear-gradient(135deg,#059669,#047857)'}}><RefreshCw size={14}/>توليد رواتب أبريل</button>
                <button onClick={()=>openModal('employee')} className="flex items-center gap-2 text-white px-4 py-2 rounded-xl font-bold text-sm hover:opacity-90" style={{background:'linear-gradient(135deg,#1d4ed8,#2563eb)'}}><Plus size={14}/>موظف</button>
              </div>
            </div>
            <div className="grid grid-cols-3 gap-4">
              <KpiCard label="إجمالي الرواتب" value={fmt(totalPayroll)} icon={Banknote} grad="from-violet-600 to-violet-400"/>
              <KpiCard label="رواتب معلقة" value={fmt(pendingPayroll)} icon={Clock} grad="from-amber-600 to-amber-400"/>
              <KpiCard label="الموظفون النشطون" value={String(employees.filter(e=>e.status==='active').length)} icon={Users} grad="from-emerald-600 to-emerald-400"/>
            </div>
            <div>
              <h4 className="text-sm font-black text-slate-300 mb-3">👥 الموظفون</h4>
              <GCard className="overflow-hidden">
                <table className="w-full text-sm">
                  <thead><tr className="border-b border-slate-800/50" style={{background:'rgba(12,20,40,0.9)'}}>
                    {['الموظف','المسمى','القسم','الراتب الأساسي','تاريخ الانضمام','الحالة','✕'].map(h=><th key={h} className="p-3 text-right text-xs font-bold text-slate-500">{h}</th>)}
                  </tr></thead>
                  <tbody>
                    {employees.map(e=>(
                      <tr key={e.id} className="hover:bg-slate-800/15 transition-colors">
                        <td className="border-b border-slate-800/40 p-3">
                          <div className="flex items-center gap-2">
                            <div className="w-7 h-7 rounded-lg flex items-center justify-center text-white font-black text-xs" style={{background:'linear-gradient(135deg,#1d4ed8,#059669)'}}>{e.name.charAt(0)}</div>
                            <span className="font-bold text-white text-sm">{e.name}</span>
                          </div>
                        </td>
                        <td className="border-b border-slate-800/40 p-3 text-slate-400 text-xs">{e.position}</td>
                        <td className="border-b border-slate-800/40 p-3"><Badge text={e.department} color="bg-blue-500/15 text-blue-400"/></td>
                        <td className="border-b border-slate-800/40 p-3 font-black text-violet-400">{fmt(e.salary)}</td>
                        <td className="border-b border-slate-800/40 p-3 text-slate-500 text-xs">{e.joinDate}</td>
                        <td className="border-b border-slate-800/40 p-3"><Badge text={e.status==='active'?'نشط':'متوقف'} color={e.status==='active'?'bg-emerald-500/15 text-emerald-400':'bg-slate-700 text-slate-500'}/></td>
                        <td className="border-b border-slate-800/40 p-3">
                          <div className="flex gap-1">
                            <button onClick={()=>openModal('employee',e)} className="p-1 text-slate-600 hover:text-blue-400 rounded transition-colors"><Edit size={12}/></button>
                            <button onClick={()=>setEmployees(employees.filter(x=>x.id!==e.id))} className="p-1 text-slate-600 hover:text-rose-400 rounded transition-colors"><Trash2 size={12}/></button>
                          </div>
                        </td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </GCard>
            </div>
            <div>
              <h4 className="text-sm font-black text-slate-300 mb-3">💰 كشف الرواتب</h4>
              <GCard className="overflow-hidden">
                <table className="w-full text-sm">
                  <thead><tr className="border-b border-slate-800/50" style={{background:'rgba(12,20,40,0.9)'}}>
                    {['الموظف','الشهر','الأساسي','مكافآت','خصومات','الصافي','الحالة','صرف'].map(h=><th key={h} className="p-3 text-right text-xs font-bold text-slate-500">{h}</th>)}
                  </tr></thead>
                  <tbody>
                    {payrolls.map(p=>(
                      <tr key={p.id} className="hover:bg-slate-800/15 transition-colors">
                        <td className="border-b border-slate-800/40 p-3 font-bold text-white text-sm">{p.employeeName}</td>
                        <td className="border-b border-slate-800/40 p-3 text-slate-400 text-xs">{p.month}</td>
                        <td className="border-b border-slate-800/40 p-3 text-slate-300">{fmt(p.baseSalary)}</td>
                        <td className="border-b border-slate-800/40 p-3 text-emerald-400">+{fmt(p.bonus)}</td>
                        <td className="border-b border-slate-800/40 p-3 text-rose-400">-{fmt(p.deductions)}</td>
                        <td className="border-b border-slate-800/40 p-3 font-black text-violet-400">{fmt(p.net)}</td>
                        <td className="border-b border-slate-800/40 p-3"><Badge text={p.status==='paid'?'مدفوع':'معلق'} color={p.status==='paid'?'bg-emerald-500/15 text-emerald-400':'bg-amber-500/15 text-amber-400'}/></td>
                        <td className="border-b border-slate-800/40 p-3">{p.status==='pending'&&<button onClick={()=>payPayroll(p.id)} className="text-xs px-2 py-1 rounded-lg font-bold text-white hover:opacity-80" style={{background:'linear-gradient(135deg,#7c3aed,#6d28d9)'}}>صرف</button>}</td>
                      </tr>
                    ))}
                  </tbody>
                  <tfoot><tr className="border-t border-slate-800" style={{background:'rgba(12,20,40,0.7)'}}>
                    <td colSpan={5} className="p-3 text-slate-500 font-bold text-xs">الإجمالي</td>
                    <td className="p-3 font-black text-violet-400 text-base">{fmt(totalPayroll)}</td>
                    <td colSpan={2}/>
                  </tr></tfoot>
                </table>
              </GCard>
            </div>
          </div>
        )}

        {/* ══ ANALYTICS ══ */}
        {tab==='analytics'&&(
          <div className="space-y-5">
            <h3 className="text-xl font-black text-white">التحليلات</h3>
            <div className="grid grid-cols-2 lg:grid-cols-4 gap-4">
              <KpiCard label="المبيعات" value={fmt(totalSales)} icon={TrendingUp} grad="from-blue-600 to-blue-400"/>
              <KpiCard label="المشتريات" value={fmt(totalPurchases)} icon={ArrowDownLeft} grad="from-emerald-600 to-emerald-400"/>
              <KpiCard label="المصروفات" value={fmt(totalExpenses)} icon={Wallet} grad="from-rose-600 to-rose-400"/>
              <KpiCard label="صافي الربح" value={fmt(netProfit)} icon={DollarSign} grad="from-violet-600 to-violet-400" trend={`${pct(netProfit,totalSales)}٪`} trendUp={netProfit>0}/>
            </div>
            <div className="grid grid-cols-1 lg:grid-cols-2 gap-5">
              <GCard className="p-5">
                <h4 className="font-black text-white mb-4 flex items-center gap-2"><PieChart size={15} className="text-blue-400"/>توزيع الإيرادات</h4>
                <div className="space-y-3">
                  {activeSales.map((s,i)=>(
                    <div key={s.id}><div className="flex justify-between mb-1">
                      <span className="text-sm text-slate-300">{s.branch}</span>
                      <span className="text-sm font-black text-white">{pct(s.total,totalSales)}٪</span>
                    </div><BarUI v={pct(s.total,totalSales)} grad={grads[i%grads.length]}/></div>
                  ))}
                </div>
              </GCard>
              <GCard className="p-5">
                <h4 className="font-black text-white mb-4 flex items-center gap-2"><Receipt size={15} className="text-amber-400"/>ملخص ضريبة القيمة المضافة</h4>
                <div className="space-y-3">
                  {[
                    {label:'ضريبة محصلة (مبيعات)',value:invoices.filter(i=>i.type==='sale').reduce((a,i)=>a+i.vatAmount,0),vc:'text-emerald-400'},
                    {label:'ضريبة مدفوعة (مشتريات)',value:invoices.filter(i=>i.type==='purchase').reduce((a,i)=>a+i.vatAmount,0),vc:'text-rose-400'},
                    {label:'صافي الضريبة المستحقة',value:invoices.filter(i=>i.type==='sale').reduce((a,i)=>a+i.vatAmount,0)-invoices.filter(i=>i.type==='purchase').reduce((a,i)=>a+i.vatAmount,0),vc:'text-blue-400'},
                  ].map(item=><Chip key={item.label} label={item.label} value={fmt(item.value)} vc={item.vc}/>)}
                </div>
                <div className="mt-4 pt-4 border-t border-slate-700/40">
                  <h5 className="text-xs font-bold text-slate-400 uppercase mb-3 tracking-wider">توزيع المصروفات</h5>
                  {expenses.map((e,i)=>(
                    <div key={e.id} className="mb-2"><div className="flex justify-between text-xs mb-1"><span className="text-slate-300">{e.category}</span><span className="text-rose-400 font-bold">{fmt(e.amount)}</span></div>
                    <BarUI v={pct(e.amount,totalExpenses)} grad={grads[i%grads.length]}/></div>
                  ))}
                </div>
              </GCard>
            </div>
          </div>
        )}

        {/* ══ OCR ══ */}
        {tab==='ocr'&&(
          <div className="space-y-4">
            <h3 className="text-xl font-black text-white flex items-center gap-2"><FileText size={19} className="text-blue-400"/>قارئ الفواتير بالذكاء الاصطناعي</h3>
            <GCard className="p-6">
              <label className="flex flex-col items-center justify-center w-full h-40 border-2 border-dashed border-slate-700/50 rounded-2xl cursor-pointer hover:border-blue-500/50 hover:bg-blue-500/4 transition-all">
                <div className="text-center"><Upload size={28} className="text-slate-600 mx-auto mb-2"/><p className="text-slate-400 font-semibold text-sm">ارفع صورة الفاتورة</p><p className="text-slate-600 text-xs mt-1">PNG, JPG, PDF</p></div>
                <input type="file" accept="image/*" onChange={handleOcr} className="hidden"/>
              </label>
              {ocrLoading&&<div className="flex items-center justify-center gap-3 mt-4 text-blue-400"><RefreshCw size={16} className="animate-spin"/><span className="font-semibold text-sm">جارٍ استخراج البيانات...</span></div>}
              {ocrFile&&!ocrLoading&&<div className="mt-4 grid grid-cols-1 lg:grid-cols-2 gap-4">
                <img src={ocrFile} alt="invoice" className="rounded-xl max-h-80 object-contain border border-slate-700/40"/>
                {ocrResult&&<div className="bg-slate-800/50 rounded-xl p-4"><p className="text-xs font-bold text-emerald-400 uppercase mb-2 tracking-wider">البيانات المستخرجة</p><p className="text-slate-300 text-sm leading-relaxed whitespace-pre-wrap">{ocrResult}</p></div>}
              </div>}
            </GCard>
          </div>
        )}

        {/* ══ REPORTS ══ */}
        {tab==='reports'&&(
          <div className="space-y-4">
            <h3 className="text-xl font-black text-white">التقارير</h3>
            <div className="grid grid-cols-1 lg:grid-cols-3 gap-4">
              {[
                {title:'تقرير شامل',sub:'المبيعات، المشتريات، المصروفات، المخزون، الرواتب',grad:'from-blue-600 to-blue-400',icon:Printer,action:printReport},
                {title:'تقرير المبيعات',sub:'مبيعات الفروع والمنتجات المباعة',grad:'from-emerald-600 to-emerald-400',icon:TrendingUp,action:printReport},
                {title:'تقرير الضريبة',sub:'ملخص ضريبة القيمة المضافة',grad:'from-amber-600 to-amber-400',icon:Receipt,action:printReport},
              ].map(r=>(
                <GCard key={r.title} onClick={r.action} glow={r.grad} className="p-6 group cursor-pointer">
                  <div className={`absolute inset-0 opacity-[0.04] group-hover:opacity-[0.08] bg-gradient-to-br ${r.grad}`}/>
                  <div className={`relative p-3 rounded-xl w-fit mb-4 bg-gradient-to-br ${r.grad}`}><r.icon size={20} className="text-white"/></div>
                  <h4 className="relative font-black text-white mb-1">{r.title}</h4>
                  <p className="relative text-slate-500 text-xs">{r.sub}</p>
                  <p className="relative text-blue-400 text-xs mt-3 font-bold">اضغط للطباعة →</p>
                </GCard>
              ))}
            </div>
          </div>
        )}

        {/* ══ USERS ══ */}
        {tab==='users'&&(
          <div className="space-y-4">
            <div className="flex justify-between items-center">
              <h3 className="text-xl font-black text-white">المستخدمون</h3>
              <button onClick={()=>openModal('user')} className="flex items-center gap-2 text-white px-4 py-2 rounded-xl font-bold text-sm hover:opacity-90" style={{background:'linear-gradient(135deg,#1d4ed8,#059669)'}}><Plus size={14}/>مستخدم</button>
            </div>
            <div className="grid grid-cols-1 lg:grid-cols-3 gap-4">
              {users.map(u=>(
                <GCard key={u.id} className="p-5">
                  <div className="flex items-start gap-3 mb-4">
                    <div className="w-10 h-10 rounded-xl flex items-center justify-center font-black text-white flex-shrink-0" style={{background:'linear-gradient(135deg,#1d4ed8,#059669)'}}>{u.avatar}</div>
                    <div className="flex-1 min-w-0">
                      <p className="font-black text-white">{u.name}</p>
                      <Badge text={u.role==='admin'?'مدير عام':u.role==='manager'?'مدير':'موظف'} color={u.role==='admin'?'bg-blue-500/15 text-blue-400':u.role==='manager'?'bg-emerald-500/15 text-emerald-400':'bg-slate-700 text-slate-400'}/>
                    </div>
                    <div className="flex gap-1">
                      <button onClick={()=>openModal('user',u)} className="p-1.5 text-slate-600 hover:text-blue-400 rounded-lg transition-colors"><Edit size={13}/></button>
                      <button onClick={()=>setUsers(prev=>prev.map(x=>x.id===u.id?{...x,active:!x.active}:x))} className={`p-1.5 rounded-lg transition-colors ${u.active?'text-emerald-400 hover:text-slate-500':'text-slate-600 hover:text-emerald-400'}`}><UserCheck size={13}/></button>
                    </div>
                  </div>
                  <div className="space-y-1.5 text-xs text-slate-500">
                    <p>{u.email}</p>
                    <p className="flex items-center gap-2"><MapPin size={11}/>{u.branch}</p>
                    <p className="text-slate-600">آخر دخول: {u.lastLogin}</p>
                  </div>
                  <div className="mt-3 pt-3 border-t border-slate-700/40 flex justify-between items-center">
                    <Badge text={u.active?'نشط':'متوقف'} color={u.active?'bg-emerald-500/15 text-emerald-400':'bg-rose-500/15 text-rose-400'}/>
                  </div>
                </GCard>
              ))}
            </div>
          </div>
        )}

        {/* ══ NOTIFICATIONS ══ */}
        {tab==='notifications'&&(
          <div className="space-y-4">
            <div className="flex justify-between items-center">
              <div><h3 className="text-xl font-black text-white">التنبيهات</h3><p className="text-slate-500 text-sm">{unread} غير مقروء</p></div>
              <button onClick={()=>setNotifications(n=>n.map(x=>({...x,read:true})))} className="text-slate-400 hover:text-white text-xs font-bold px-3 py-2 rounded-xl hover:bg-slate-800 transition-colors border border-slate-700/40">قراءة الكل</button>
            </div>
            <div className="space-y-3">
              {notifications.map(n=>(
                <GCard key={n.id} className={`p-4 transition-all ${!n.read?'border-slate-700/60':''}`} style={!n.read?{background:'rgba(12,18,35,0.95)'}:{}}>
                  <div className="flex items-start gap-3">
                    <div className={`p-2 rounded-xl flex-shrink-0 ${n.type==='warning'?'bg-amber-500/15 text-amber-400':n.type==='success'?'bg-emerald-500/15 text-emerald-400':n.type==='error'?'bg-rose-500/15 text-rose-400':'bg-blue-500/15 text-blue-400'}`}>
                      {n.type==='warning'?<AlertTriangle size={14}/>:n.type==='success'?<CheckCircle2 size={14}/>:n.type==='error'?<AlertCircle size={14}/>:<Activity size={14}/>}
                    </div>
                    <div className="flex-1">
                      <div className="flex justify-between items-start">
                        <p className={`font-bold text-sm ${!n.read?'text-white':'text-slate-400'}`}>{n.title}</p>
                        <span className="text-xs text-slate-600">{n.time}</span>
                      </div>
                      <p className="text-xs text-slate-500 mt-0.5">{n.message}</p>
                    </div>
                    {!n.read&&<div className="w-2 h-2 bg-blue-500 rounded-full mt-1 flex-shrink-0"/>}
                  </div>
                </GCard>
              ))}
            </div>
          </div>
        )}

      </div>

      {/* ══ AI CHAT PANEL ══ */}
      {chatOpen&&(
        <div className="w-80 flex-shrink-0 border-r border-slate-800/40 flex flex-col" style={{background:'rgba(5,9,20,0.98)'}}>
          <div className="p-4 border-b border-slate-800/40 flex justify-between items-center">
            <div className="flex items-center gap-2">
              <div className="p-1.5 rounded-xl" style={{background:'linear-gradient(135deg,#1d4ed8,#059669)'}}><Brain size={14} className="text-white"/></div>
              <div><p className="text-sm font-black text-white">مساعد AI</p><p className="text-xs text-emerald-400">متصل</p></div>
            </div>
            <button onClick={()=>setChatOpen(false)} className="p-1 text-slate-600 hover:text-slate-400 rounded-lg"><X size={14}/></button>
          </div>
          <div className="flex-1 overflow-y-auto p-4 space-y-3">
            {chatMsgs.map((m,i)=>(
              <div key={i} className={`flex ${m.role==='user'?'justify-start':'justify-end'}`}>
                <div className={`max-w-[85%] rounded-2xl px-3 py-2.5 text-xs leading-relaxed ${m.role==='user'?'bg-blue-500/15 text-blue-100 rounded-tr-sm':'bg-slate-800/70 text-slate-300 rounded-tl-sm'}`}>
                  {m.content}
                </div>
              </div>
            ))}
            {chatLoading&&<div className="flex justify-end"><div className="bg-slate-800/70 rounded-2xl rounded-tl-sm px-3 py-2.5 flex items-center gap-1">{[0,1,2].map(i=><div key={i} className="w-1.5 h-1.5 bg-slate-500 rounded-full animate-bounce" style={{animationDelay:`${i*0.15}s`}}/>)}</div></div>}
            <div ref={chatEndRef}/>
          </div>
          <div className="p-3 border-t border-slate-800/40">
            <div className="flex gap-2">
              <input value={chatInput} onChange={e=>setChatInput(e.target.value)} onKeyDown={e=>e.key==='Enter'&&sendChat()} placeholder="اسألني..." className="flex-1 bg-slate-800/70 border border-slate-700/50 rounded-xl px-3 py-2 text-white text-xs outline-none placeholder-slate-600 text-right"/>
              <button onClick={sendChat} disabled={chatLoading} className="p-2 rounded-xl text-white disabled:opacity-50" style={{background:'linear-gradient(135deg,#1d4ed8,#059669)'}}><Send size={13}/></button>
            </div>
          </div>
        </div>
      )}
    </div>
  </main>

  {/* ══ MODALS ══ */}
  {modal==='product'&&(
    <Modal title={editTarget?'تعديل منتج':'منتج جديد'} onClose={closeModal} wide icon={Package} ic="text-emerald-400">
      <div className="grid grid-cols-2 gap-4">
        <Field label="اسم المنتج" req><Inp value={form.name||''} onChange={v=>setForm(f=>({...f,name:v}))} ph="اسم المنتج"/></Field>
        <Field label="وحدة القياس"><Inp value={form.unit||''} onChange={v=>setForm(f=>({...f,unit:v}))} ph="جهاز / قطعة / كيلو"/></Field>
        <div className="col-span-2"><Field label="وصف المنتج"><textarea value={form.description||''} onChange={e=>setForm(f=>({...f,description:e.target.value}))} rows={2} placeholder="وصف مختصر..." className="w-full bg-slate-800/70 border border-slate-700/50 rounded-xl px-3.5 py-2.5 text-white text-sm outline-none text-right placeholder-slate-600 resize-none"/></Field></div>
        <Field label="الباركود"><Inp value={form.barcode||''} onChange={v=>setForm(f=>({...f,barcode:v}))} ph="BAR-001"/></Field>
        <Field label="التصنيف"><Inp value={form.category||''} onChange={v=>setForm(f=>({...f,category:v}))} ph="إلكترونيات"/></Field>
        <Field label="السعر قبل الضريبة" req><Inp value={form.unitPrice||''} onChange={v=>{const p=parseFloat(v)||0;const vat=parseFloat(form.vatRate)||VAT_RATE_DEFAULT;setForm(f=>({...f,unitPrice:v,priceAfterVat:String(parseFloat(calcAfterVat(p,vat).toFixed(2)))}));}} type="number" ph="0.00"/></Field>
        <Field label="نسبة الضريبة ٪"><Inp value={form.vatRate||String(VAT_RATE_DEFAULT)} onChange={v=>{const vat=parseFloat(v)||0;const p=parseFloat(form.unitPrice)||0;setForm(f=>({...f,vatRate:v,priceAfterVat:String(parseFloat(calcAfterVat(p,vat).toFixed(2)))}));}} type="number" ph="15"/></Field>
        <Field label="السعر بعد الضريبة"><Inp value={form.priceAfterVat||''} onChange={v=>{const av=parseFloat(v)||0;const vat=parseFloat(form.vatRate)||VAT_RATE_DEFAULT;setForm(f=>({...f,priceAfterVat:v,unitPrice:String(parseFloat((av/(1+vat/100)).toFixed(2)))}));}} type="number" ph="0.00"/></Field>
        <Field label="المخزون الحالي"><Inp value={form.stock||''} onChange={v=>setForm(f=>({...f,stock:v}))} type="number" ph="0"/></Field>
        <Field label="الحد الأدنى"><Inp value={form.minStock||''} onChange={v=>setForm(f=>({...f,minStock:v}))} type="number" ph="5"/></Field>
      </div>
      <div className="flex gap-3 mt-5">
        <BtnP onClick={saveProduct} g="#059669,#047857" full><CheckCircle2 size={14}/>{editTarget?'تحديث':'إضافة'}</BtnP>
        <BtnP onClick={closeModal} g="#475569,#334155" full>إلغاء</BtnP>
      </div>
    </Modal>
  )}
  {modal==='user'&&(
    <Modal title={editTarget?'تعديل مستخدم':'مستخدم جديد'} onClose={closeModal} icon={Shield} ic="text-blue-400">
      <div className="space-y-3">
        <Field label="الاسم" req><Inp value={form.name||''} onChange={v=>setForm(f=>({...f,name:v}))} ph="الاسم الكامل"/></Field>
        <Field label="البريد الإلكتروني" req><Inp value={form.email||''} onChange={v=>setForm(f=>({...f,email:v}))} type="email" ph="email@example.com"/></Field>
        <Field label="الدور"><Sel value={form.role||'staff'} onChange={v=>setForm(f=>({...f,role:v}))}><option value="admin">مدير عام</option><option value="manager">مدير</option><option value="staff">موظف</option></Sel></Field>
        <Field label="الفرع"><Inp value={form.branch||''} onChange={v=>setForm(f=>({...f,branch:v}))} ph="اسم الفرع"/></Field>
      </div>
      <div className="flex gap-3 mt-5"><BtnP onClick={saveUser} g="#1d4ed8,#2563eb" full><CheckCircle2 size={14}/>{editTarget?'تحديث':'إضافة'}</BtnP><BtnP onClick={closeModal} g="#475569,#334155" full>إلغاء</BtnP></div>
    </Modal>
  )}
  {modal==='customer'&&(
    <Modal title={editTarget?'تعديل عميل':'عميل جديد'} onClose={closeModal} icon={Users} ic="text-blue-400">
      <div className="space-y-3">
        <Field label="الاسم" req><Inp value={form.name||''} onChange={v=>setForm(f=>({...f,name:v}))} ph="اسم العميل"/></Field>
        <div className="grid grid-cols-2 gap-3">
          <Field label="الهاتف"><Inp value={form.phone||''} onChange={v=>setForm(f=>({...f,phone:v}))} ph="0500-000-000"/></Field>
          <Field label="البريد"><Inp value={form.email||''} onChange={v=>setForm(f=>({...f,email:v}))} type="email"/></Field>
        </div>
        <Field label="العنوان"><Inp value={form.address||''} onChange={v=>setForm(f=>({...f,address:v}))} ph="المدينة"/></Field>
        <Field label="النوع"><Sel value={form.type||'retail'} onChange={v=>setForm(f=>({...f,type:v}))}><option value="retail">تجزئة</option><option value="wholesale">جملة</option></Sel></Field>
        <Field label="ملاحظات"><Inp value={form.notes||''} onChange={v=>setForm(f=>({...f,notes:v}))} ph="ملاحظات..."/></Field>
      </div>
      <div className="flex gap-3 mt-5"><BtnP onClick={saveCustomer} g="#1d4ed8,#2563eb" full><CheckCircle2 size={14}/>{editTarget?'تحديث':'إضافة'}</BtnP><BtnP onClick={closeModal} g="#475569,#334155" full>إلغاء</BtnP></div>
    </Modal>
  )}
  {modal==='supplier'&&(
    <Modal title={editTarget?'تعديل مورد':'مورد جديد'} onClose={closeModal} icon={Truck} ic="text-emerald-400">
      <div className="space-y-3">
        <Field label="الاسم" req><Inp value={form.name||''} onChange={v=>setForm(f=>({...f,name:v}))} ph="اسم المورد"/></Field>
        <div className="grid grid-cols-2 gap-3">
          <Field label="الهاتف"><Inp value={form.phone||''} onChange={v=>setForm(f=>({...f,phone:v}))} ph="011-000-0000"/></Field>
          <Field label="البريد"><Inp value={form.email||''} onChange={v=>setForm(f=>({...f,email:v}))} type="email"/></Field>
        </div>
        <Field label="العنوان"><Inp value={form.address||''} onChange={v=>setForm(f=>({...f,address:v}))} ph="المدينة"/></Field>
        <Field label="التصنيف"><Inp value={form.category||''} onChange={v=>setForm(f=>({...f,category:v}))} ph="إلكترونيات / مستلزمات..."/></Field>
      </div>
      <div className="flex gap-3 mt-5"><BtnP onClick={saveSupplier} g="#059669,#047857" full><CheckCircle2 size={14}/>{editTarget?'تحديث':'إضافة'}</BtnP><BtnP onClick={closeModal} g="#475569,#334155" full>إلغاء</BtnP></div>
    </Modal>
  )}
  {modal==='invoice'&&(
    <Modal title="فاتورة جديدة" onClose={closeModal} wide icon={Receipt} ic="text-blue-400">
      <div className="grid grid-cols-2 gap-4 mb-4">
        <Field label="الطرف (عميل/مورد)" req><Inp value={form.party||''} onChange={v=>setForm(f=>({...f,party:v}))} ph="اسم العميل أو المورد"/></Field>
        <Field label="النوع"><Sel value={form.type||'sale'} onChange={v=>setForm(f=>({...f,type:v}))}><option value="sale">فاتورة بيع</option><option value="purchase">فاتورة شراء</option></Sel></Field>
        <Field label="التاريخ"><Inp value={form.date||today()} onChange={v=>setForm(f=>({...f,date:v}))} type="date"/></Field>
      </div>
      <div className="mb-4">
        <div className="flex justify-between items-center mb-2"><p className="text-xs font-bold text-slate-400 uppercase tracking-wider">البنود</p><button onClick={()=>setInvItems(p=>[...p,{name:'',qty:1,price:0,vatRate:VAT_RATE_DEFAULT}])} className="text-xs text-blue-400 font-bold hover:text-blue-300"><Plus size={12} className="inline ml-1"/>بند</button></div>
        {invItems.map((item,i)=>(
          <div key={i} className="grid grid-cols-5 gap-2 mb-2">
            <div className="col-span-2"><Inp value={item.name} onChange={v=>setInvItems(p=>p.map((x,j)=>j===i?{...x,name:v}:x))} ph="البند"/></div>
            <Inp value={String(item.qty)} onChange={v=>setInvItems(p=>p.map((x,j)=>j===i?{...x,qty:parseFloat(v)||0}:x))} type="number" ph="كمية"/>
            <Inp value={String(item.price)} onChange={v=>setInvItems(p=>p.map((x,j)=>j===i?{...x,price:parseFloat(v)||0}:x))} type="number" ph="سعر"/>
            <div className="flex gap-1">
              <Inp value={String(item.vatRate)} onChange={v=>setInvItems(p=>p.map((x,j)=>j===i?{...x,vatRate:parseFloat(v)||0}:x))} type="number" ph="ض٪"/>
              <button onClick={()=>setInvItems(p=>p.filter((_,j)=>j!==i))} className="text-rose-400 hover:text-rose-300 p-1"><Trash2 size={14}/></button>
            </div>
          </div>
        ))}
        <div className="bg-slate-800/40 rounded-xl p-3 text-xs mt-2">
          <div className="flex justify-between mb-1"><span className="text-slate-400">قبل الضريبة:</span><span className="text-white font-bold">{fmt(invItems.reduce((a,i)=>a+(i.price*i.qty),0))}</span></div>
          <div className="flex justify-between mb-1"><span className="text-amber-400">الضريبة:</span><span className="text-amber-400 font-bold">{fmt(invItems.reduce((a,i)=>a+(calcVat(i.price,i.vatRate)*i.qty),0))}</span></div>
          <div className="flex justify-between font-black border-t border-slate-700/40 pt-1"><span className="text-white">الإجمالي:</span><span className="text-blue-400">{fmt(invItems.reduce((a,i)=>a+(i.price*i.qty),0)+invItems.reduce((a,i)=>a+(calcVat(i.price,i.vatRate)*i.qty),0))}</span></div>
        </div>
      </div>
      <div className="flex gap-3"><BtnP onClick={saveInvoice} g="#1d4ed8,#059669" full><CheckCircle2 size={14}/>إنشاء الفاتورة</BtnP><BtnP onClick={closeModal} g="#475569,#334155" full>إلغاء</BtnP></div>
    </Modal>
  )}
  {modal==='employee'&&(
    <Modal title={editTarget?'تعديل موظف':'موظف جديد'} onClose={closeModal} icon={Users} ic="text-violet-400">
      <div className="space-y-3">
        <Field label="الاسم" req><Inp value={form.name||''} onChange={v=>setForm(f=>({...f,name:v}))} ph="اسم الموظف الكامل"/></Field>
        <div className="grid grid-cols-2 gap-3">
          <Field label="المسمى الوظيفي"><Inp value={form.position||''} onChange={v=>setForm(f=>({...f,position:v}))} ph="مدير مبيعات"/></Field>
          <Field label="القسم"><Inp value={form.department||''} onChange={v=>setForm(f=>({...f,department:v}))} ph="المبيعات"/></Field>
        </div>
        <div className="grid grid-cols-2 gap-3">
          <Field label="الراتب الأساسي" req><Inp value={form.salary||''} onChange={v=>setForm(f=>({...f,salary:v}))} type="number" ph="0"/></Field>
          <Field label="تاريخ الانضمام"><Inp value={form.joinDate||''} onChange={v=>setForm(f=>({...f,joinDate:v}))} type="date"/></Field>
        </div>
        <div className="grid grid-cols-2 gap-3">
          <Field label="الهاتف"><Inp value={form.phone||''} onChange={v=>setForm(f=>({...f,phone:v}))} ph="0500-000-000"/></Field>
          <Field label="البريد"><Inp value={form.email||''} onChange={v=>setForm(f=>({...f,email:v}))} type="email" ph="email@example.com"/></Field>
        </div>
      </div>
      <div className="flex gap-3 mt-5"><BtnP onClick={saveEmployee} g="#7c3aed,#6d28d9" full><CheckCircle2 size={14}/>{editTarget?'تحديث':'إضافة'}</BtnP><BtnP onClick={closeModal} g="#475569,#334155" full>إلغاء</BtnP></div>
    </Modal>
  )}
</div>
```

);
}