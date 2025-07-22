# nomivalle
# Firebase Studio

This is a NextJS starter in Firebase Studio.

To get started, take a look at src/app/page.tsx.
'use client';
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from '@/components/ui/card';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Button } from '@/components/ui/button';
import { FileDown, Share2, User } from 'lucide-react';
import { Badge } from '@/components/ui/badge';
import jsPDF from 'jspdf';
import 'jspdf-autotable';
import { useToast } from '@/hooks/use-toast';

const accessLog = [
    { id: 1, adminName: 'Admin Principal', ipAddress: '192.168.1.10', timestamp: '2024-07-29 09:15:23', status: 'Éxito' },
    { id: 2, adminName: 'Contador', ipAddress: '201.248.50.12', timestamp: '2024-07-29 09:30:11', status: 'Éxito' },
    { id: 3, adminName: 'Desconocido', ipAddress: '180.50.11.2', timestamp: '2024-07-29 10:05:01', status: 'Fallido' },
    { id: 4, adminName: 'Admin Principal', ipAddress: '192.168.1.10', timestamp: '2024-07-28 15:45:50', status: 'Éxito' },
];

export default function AccessReportPage() {
  const { toast } = useToast();

  const exportToPDF = async () => {
    const doc = new jsPDF();
    doc.text("Reporte de Accesos al Sistema", 14, 15);
    (doc as any).autoTable({
      head: [['Administrador', 'Dirección IP', 'Fecha y Hora', 'Estado']],
      body: accessLog.map(log => [
        log.adminName,
        log.ipAddress,
        log.timestamp,
        log.status
      ]),
      startY: 20
    });
    
    const pdfBlob = doc.output('blob');
    const fileName = 'reporte_accesos.pdf';
    const pdfFile = new File([pdfBlob], fileName, { type: 'application/pdf' });

    if (navigator.canShare && navigator.canShare({ files: [pdfFile] })) {
      try {
        await navigator.share({
          files: [pdfFile],
          title: 'Reporte de Accesos',
          text: 'Aquí está el reporte de accesos al sistema.',
        });
        toast({ title: 'Éxito', description: 'Reporte compartido.' });
      } catch (error) {
        if ((error as Error).name !== 'AbortError') {
          console.error('Error al compartir:', error);
          doc.save(fileName);
          toast({ variant: 'destructive', title: 'Error al Compartir', description: 'El reporte se ha descargado en su lugar.' });
        }
      }
    } else {
      doc.save(fileName);
      toast({ title: 'Reporte Descargado', description: 'El reporte de accesos ha sido descargado.' });
    }
  };

  const getStatusVariant = (status: string) => {
    return status === 'Éxito' ? 'default' : 'destructive';
  };

  return (
    <Card>
      <CardHeader className="flex flex-row items-center justify-between">
        <div>
          <CardTitle>Reporte de Accesos al Sistema</CardTitle>
          <CardDescription>
            Historial de inicios de sesión de los administradores.
          </CardDescription>
        </div>
        <Button onClick={exportToPDF} variant="outline" size="sm">
            <Share2 className="mr-2 h-4 w-4"/>
            Compartir o Exportar
        </Button>
      </CardHeader>
      <CardContent>
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Administrador</TableHead>
              <TableHead>Dirección IP</TableHead>
              <TableHead>Fecha y Hora</TableHead>
              <TableHead className="text-right">Estado</TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {accessLog.map((log) => (
              <TableRow key={log.id}>
                <TableCell className="font-medium flex items-center gap-2"><User className="h-4 w-4 text-muted-foreground"/>{log.adminName}</TableCell>
                <TableCell>{log.ipAddress}</TableCell>
                <TableCell>{log.timestamp}</TableCell>
                <TableCell className="text-right">
                    <Badge variant={getStatusVariant(log.status)}>{log.status}</Badge>
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </CardContent>
    </Card>
  );
}
// =================================================================================
// FILE: src/app/dashboard/calculations/page.tsx
'use client';
import { useState, useEffect } from 'react';
import { Button } from '@/components/ui/button';
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from '@/components/ui/card';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@/components/ui/dialog';
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from '@/components/ui/alert-dialog';
import { Input } from '@/components/ui/input';
import { RadioGroup, RadioGroupItem } from '@/components/ui/radio-group';
import { employees as initialEmployees, bcvRate as defaultBcvRate, minimumWageBs, Employee, companyName as defaultCompanyName, companyRif as defaultCompanyRif, companyAddress as defaultCompanyAddress, paymentConcepts as initialPaymentConcepts, deductionConcepts as initialDeductionConcepts, Adjustment, Concept } from '@/lib/placeholder-data';
import { Label } from '@/components/ui/label';
import { FileDown, MoreVertical, PlusCircle, ArrowLeft, ArrowRight, User, Briefcase, Plane, Gift, CalculatorIcon, Share2, Download, Save, Trash2 } from 'lucide-react';
import { Separator } from '@/components/ui/separator';
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger, DropdownMenuSeparator, DropdownMenuLabel } from '@/components/ui/dropdown-menu';
import { useToast } from '@/hooks/use-toast';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { differenceInYears, differenceInMonths, format, parseISO, lastDayOfMonth } from 'date-fns';
import { es } from 'date-fns/locale';
import jsPDF from 'jspdf';
import 'jspdf-autotable';
import * as XLSX from 'xlsx';
import { saveAs } from 'file-saver';
import { Receipt, ReportFooter, ReportHeader, getReceiptData } from '@/components/receipt';


const formatCurrency = (amount: number, currency: 'USD' | 'VES') => {
  return new Intl.NumberFormat('es-VE', {
    style: 'currency',
    currency: currency,
    minimumFractionDigits: 2,
  }).format(amount);
};

// --- Helper Functions ---
const calculateSeniority = (hireDate: string, terminationDate: string) => {
    const start = parseISO(hireDate);
    const end = parseISO(terminationDate);
    const years = differenceInYears(end, start);
    const months = differenceInMonths(end, start) % 12;
    return { years, months, totalMonths: differenceInYears(end, start) * 12 + differenceInMonths(end, start) % 12 };
};

const calculateIntegralSalary = (employee: Employee, activePaymentConcepts: Concept[], bcvRate: number) => {
    // Calculate normal monthly salary including all recurring payments
    let normalMonthlySalary = employee.baseSalaryBs;
    activePaymentConcepts.forEach(concept => {
        if (concept.active) { 
             let conceptAmountVes = 0;
             if (concept.amountType === 'USD') {
                 conceptAmountVes = (concept.amountUsd || 0) * bcvRate;
             } else {
                 conceptAmountVes = concept.amountVes || 0;
             }
             normalMonthlySalary += conceptAmountVes;
        }
    });

    const dailyNormalSalary = normalMonthlySalary / 30;
    
    // Aliquot of Utilidades (Art. 131 LOTTT - min 30 days)
    const dailyBonusUtilidades = (normalMonthlySalary * (30/30)) / 360;

    // Aliquot of Vacation Bonus (Art. 192 LOTTT - min 15 days)
    const seniorityYears = differenceInYears(new Date(), parseISO(employee.hireDate));
    const vacationDays = 15 + (seniorityYears > 1 ? seniorityYears - 1 : 0);
    const dailyBonusVacacional = (dailyNormalSalary * vacationDays) / 360;

    const dailyIntegralSalary = dailyNormalSalary + dailyBonusUtilidades + dailyBonusVacacional;
    
    return {
        dailyIntegralSalary,
        monthlyIntegralSalary: dailyIntegralSalary * 30,
        dailyNormalSalary,
        normalMonthlySalary,
    };
};

function PayrollCalculationTab() {
  const { toast } = useToast();
  const [employees, setEmployees] = useState<Employee[]>([]);
  const [bcvRate, setBcvRate] = useState(defaultBcvRate);
  
  const [activePaymentConcepts, setActivePaymentConcepts] = useState<Concept[]>([]);
  const [activeDeductionConcepts, setActiveDeductionConcepts] = useState<Concept[]>([]);
  
  const [payrollPeriod, setPayrollPeriod] = useState({
    year: new Date().getFullYear().toString(),
    month: format(new Date(), 'MMMM', { locale: es }),
    fortnight: 'first' as 'first' | 'second',
  });
  
  const [adjustments, setAdjustments] = useState<Adjustment[]>([]);

  useEffect(() => {
    const loadData = () => {
        const savedEmployees = localStorage.getItem('employees');
        const savedBcvRate = localStorage.getItem('bcvRate');
        const savedPaymentConcepts = localStorage.getItem('paymentConcepts');
        const savedDeductionConcepts = localStorage.getItem('deductionConcepts');
        
        const savedYear = localStorage.getItem('payrollYear');
        const savedMonth = localStorage.getItem('payrollMonth');
        const savedFortnight = localStorage.getItem('payrollFortnight');
        
        const savedAdjustments = localStorage.getItem('payrollAdjustments');

        setEmployees(savedEmployees ? JSON.parse(savedEmployees) : initialEmployees);
        setBcvRate(savedBcvRate ? parseFloat(savedBcvRate) : defaultBcvRate);
        
        const allPaymentConcepts = savedPaymentConcepts ? JSON.parse(savedPaymentConcepts) : initialPaymentConcepts;
        setActivePaymentConcepts(allPaymentConcepts.filter((c: Concept) => c.active));
        const allDeductionConcepts = savedDeductionConcepts ? JSON.parse(savedDeductionConcepts) : initialDeductionConcepts;
        setActiveDeductionConcepts(allDeductionConcepts.filter((c: Concept) => c.active));
        
        setAdjustments(savedAdjustments ? JSON.parse(savedAdjustments) : []);

        setPayrollPeriod({
            year: savedYear || new Date().getFullYear().toString(),
            month: savedMonth || format(new Date(), 'MMMM', { locale: es }),
            fortnight: (savedFortnight as 'first' | 'second') || 'first',
        });
    };

    loadData();

    window.addEventListener('storage', loadData);

    return () => {
        window.removeEventListener('storage', loadData);
    };
  }, []);
  
  const activeEmployees = employees.filter(e => e.status === 'Activo');
  const [currentEmployeeIndex, setCurrentEmployeeIndex] = useState(0);
  const selectedEmployee = activeEmployees.length > 0 ? activeEmployees[currentEmployeeIndex] : null;
  
  const [concept, setConcept] = useState('manual');
  const [description, setDescription] = useState('');
  const [amount, setAmount] = useState('');
  const [type, setType] = useState<'assignment' | 'deduction'>('assignment');
  const [adjustmentCurrency, setAdjustmentCurrency] = useState<'VES' | 'USD'>('VES');
  const [isDialogOpen, setIsDialogOpen] = useState(false);
  const [isProcessing, setIsProcessing] = useState(false);

  const [companyName, setCompanyName] = useState(defaultCompanyName);
  const [companyRif, setCompanyRif] = useState(defaultCompanyRif);
  const [companyAddress, setCompanyAddress] = useState(defaultCompanyAddress);
  
  useEffect(() => {
    const loadCompanyInfo = () => {
        setCompanyName(localStorage.getItem('companyName') || defaultCompanyName);
        setCompanyRif(localStorage.getItem('companyRif') || defaultCompanyRif);
        setCompanyAddress(localStorage.getItem('companyAddress') || defaultCompanyAddress);
    };

    loadCompanyInfo();

    window.addEventListener('storage', loadCompanyInfo);

    return () => {
        window.removeEventListener('storage', loadCompanyInfo);
    };
  }, []);
  
  useEffect(() => {
      if (currentEmployeeIndex >= activeEmployees.length) {
          setCurrentEmployeeIndex(0);
      }
  }, [activeEmployees.length, currentEmployeeIndex]);

  const handleAddAdjustment = () => {
    let finalDescription = description;
    let conceptId = `manual-${Date.now()}`;
    let conceptToastName = description;

    if (concept !== 'manual') {
        const conceptList = type === 'assignment' ? activePaymentConcepts : activeDeductionConcepts;
        const selectedConcept = conceptList.find(c => c.id === concept);
        if (selectedConcept) {
            finalDescription = selectedConcept.name;
            conceptId = selectedConcept.id;
            conceptToastName = selectedConcept.name;
        }
    }

    const numericAmount = parseFloat(amount);
    if (!selectedEmployee || !finalDescription || !amount || isNaN(numericAmount)) {
        toast({
            variant: 'destructive',
            title: 'Error de Validación',
            description: 'Por favor, complete todos los campos correctamente.',
        });
      return;
    }
    
    const amountInVes = adjustmentCurrency === 'USD' ? numericAmount * bcvRate : numericAmount;

    const newAdjustment: Adjustment = {
      employeeCedula: selectedEmployee.cedula,
      conceptId: conceptId,
      description: finalDescription,
      amount: amountInVes,
      type,
    };

    // If an adjustment for this conceptId already exists, replace it. Otherwise, add it.
    const existingAdjustmentIndex = adjustments.findIndex(
      adj => adj.employeeCedula === newAdjustment.employeeCedula && adj.conceptId === newAdjustment.conceptId
    );
    
    let updatedAdjustments;
    if (existingAdjustmentIndex > -1) {
        updatedAdjustments = [...adjustments];
        updatedAdjustments[existingAdjustmentIndex] = newAdjustment;
        toast({ title: 'Ajuste Actualizado', description: `Se actualizó el monto para "${conceptToastName}" a ${formatCurrency(amountInVes, 'VES')}.` });
    } else {
        updatedAdjustments = [...adjustments, newAdjustment];
        toast({ title: 'Ajuste Agregado', description: `Se añadió "${conceptToastName}" por ${formatCurrency(amountInVes, 'VES')}.` });
    }
    setAdjustments(updatedAdjustments);
    
    // Reset form
    setConcept('manual');
    setDescription('');
    setAmount('');
    setType('assignment');
    setAdjustmentCurrency('VES');
    setIsDialogOpen(false);
  };
  
  const generatePdfForEmployee = (doc: jsPDF, employee: Employee) => {
    const pageWidth = doc.internal.pageSize.getWidth();
    const pageMargin = 15;
    
    const receiptData = getReceiptData({
        employee,
        adjustments,
        bcvRate,
        activePaymentConcepts,
        activeDeductionConcepts,
        payrollPeriod,
    });

    let finalY = 15;

    ReportHeader({ companyName, companyRif, companyAddress }).props.children.forEach((line: any) => {
        if (typeof line === 'object' && line.props) {
            doc.setFontSize(line.props.className.includes('text-lg') ? 14 : 10);
            doc.setFont('helvetica', line.props.className.includes('font-bold') ? 'bold' : 'normal');
            doc.text(line.props.children, pageWidth / 2, finalY, { align: 'center' });
            finalY += line.props.className.includes('text-lg') ? 6 : 4;
        }
    });
    finalY += 4;

    doc.setFontSize(12); doc.setFont('helvetica', 'bold');
    doc.text('Recibo de Pago Quincenal', pageMargin, finalY);
    doc.setFontSize(10); doc.setFont('helvetica', 'normal');
    doc.text(`${employee.name} | C.I. ${employee.cedula}`, pageWidth - pageMargin, finalY, { align: 'right' });
    finalY += 6;
    doc.setTextColor(100);
    doc.text(`${receiptData.periodString} | Tasa BCV: ${formatCurrency(bcvRate, 'VES')}`, pageMargin, finalY); finalY += 6;
    doc.setTextColor(0);
    
    (doc as any).autoTable({
      startY: finalY,
      theme: 'plain',
      styles: { fontSize: 9, cellPadding: { top: 0, bottom: 0 } },
      body: [[`Cargo: ${employee.position}`, {content: `Fecha Ingreso: ${format(parseISO(employee.hireDate), 'dd/MM/yyyy')}`, styles: { halign: 'right' }}]]
    });
    finalY = (doc as any).lastAutoTable.finalY + 5;

    const assignmentsBody = receiptData.assignmentsList.map(item => [
        { content: `${item.description}${item.note ? `\n${item.note}` : ''}`, styles: { textColor: item.note ? [100, 100, 100] : [0,0,0], cellPadding: {top: 1, bottom: 1} } },
        { content: formatCurrency(item.amountBs, 'VES'), styles: { halign: 'right' } }
    ]);
    const deductionsBody = receiptData.deductionsList.map(item => [
         item.description,
        { content: formatCurrency(-item.amountBs, 'VES'), styles: { halign: 'right' } }
    ]);
    
    (doc as any).autoTable({
        startY: finalY,
        theme: 'grid',
        headStyles: { fillColor: [240, 240, 240], textColor: [0,0,0], fontStyle: 'bold' },
        columnStyles: { 0: {cellWidth: 80}, 1: { halign: 'right' }, 2: {cellWidth: 80}, 3: { halign: 'right' } },
        head: [['Asignaciones', 'Monto (VES)'], ['Deducciones', 'Monto (VES)']],
        body: assignmentsBody.map((row, i) => [...row, ...(deductionsBody[i] || ['', ''])]),
    });
    finalY = (doc as any).lastAutoTable.finalY + 5;

    (doc as any).autoTable({
        startY: finalY,
        theme: 'plain',
        styles: { fontStyle: 'bold' },
        body: [
            ['Total Asignaciones:', { content: formatCurrency(receiptData.totalAssignmentsBs, 'VES'), styles: { halign: 'right' } }],
            ['Total Deducciones:', { content: formatCurrency(-receiptData.totalDeductionsBs, 'VES'), styles: { halign: 'right' } }],
        ]
    });
    finalY = (doc as any).lastAutoTable.finalY + 2;

    (doc as any).autoTable({
        startY: finalY,
        theme: 'grid',
        headStyles: { fillColor: [220, 220, 220] },
        body: [[
            { content: 'NETO A PAGAR', styles: { fontStyle: 'bold', fontSize: 12, valign: 'middle' } },
            { content: `${formatCurrency(receiptData.netToPayBs, 'VES')}\n${formatCurrency(receiptData.netToPayBs / bcvRate, 'USD')}`, styles: { halign: 'right', fontStyle: 'bold', fontSize: 12 } },
        ]]
    });
    finalY = (doc as any).lastAutoTable.finalY + 20;

    doc.line(pageMargin + 10, finalY, pageMargin + 70, finalY);
    doc.text('Firma del Trabajador', pageMargin + 40, finalY + 5, { align: 'center' });
    doc.text(employee.name, pageMargin + 40, finalY + 10, { align: 'center' });
    doc.text(`C.I. ${employee.cedula}`, pageMargin + 40, finalY + 15, { align: 'center' });
    doc.line(pageWidth - 70, finalY, pageWidth - pageMargin - 10, finalY);
    doc.text('Firma del Empleador', pageWidth - 40, finalY + 5, { align: 'center' });
    doc.text(companyName, pageWidth - 40, finalY + 10, { align: 'center' });
    doc.text(`RIF: ${companyRif}`, pageWidth - 40, finalY + 15, { align: 'center' });
  }

  const handleProcessPayroll = async () => {
    setIsProcessing(true);
    toast({ title: "Procesando Nómina...", description: "Generando el archivo PDF de todos los recibos." });

    const doc = new jsPDF({ orientation: 'p', unit: 'pt', format: 'letter' });
    
    activeEmployees.forEach((employee, index) => {
        if (index > 0) doc.addPage();
        generatePdfForEmployee(doc, employee);
    });

    try {
        const pdfBlob = doc.output('blob');
        const fileName = 'nomina_quincenal.pdf';
        const pdfFile = new File([pdfBlob], fileName, { type: 'application/pdf' });

        if (navigator.canShare && navigator.canShare({ files: [pdfFile] })) {
            await navigator.share({
                files: [pdfFile],
                title: 'Nómina Quincenal',
                text: 'Se ha generado la nómina quincenal con todos los recibos.',
            });
            toast({ title: 'Éxito', description: 'Nómina compartida.' });
        } else {
            doc.save(fileName);
            toast({ title: 'Nómina Descargada', description: 'El archivo con todos los recibos ha sido descargado.' });
        }
    } catch (error) {
        console.error('Error procesando la nómina:', error);
        if ((error as Error).name !== 'AbortError') {
          doc.save('nomina_quincenal.pdf');
          toast({ variant: 'destructive', title: 'Error al Compartir', description: 'La nómina se ha descargado en su lugar.' });
        }
    } finally {
        setIsProcessing(false);
    }
  };

  const handleDownloadIndividualReceipt = () => {
      if (!selectedEmployee) return;

      const doc = new jsPDF({ orientation: 'p', unit: 'pt', format: 'letter' });
      generatePdfForEmployee(doc, selectedEmployee);
      doc.save(`recibo_${selectedEmployee.cedula}.pdf`);
      toast({ title: 'Recibo Descargado', description: `El recibo de ${selectedEmployee.name} ha sido descargado.` });
  };
  
  const handleSaveAdjustments = () => {
    localStorage.setItem('payrollAdjustments', JSON.stringify(adjustments));
    toast({ title: 'Ajustes Guardados', description: 'Los ajustes individuales han sido guardados exitosamente.' });
  };

  const handleClearAdjustments = () => {
      if (!selectedEmployee) return;
      const updatedAdjustments = adjustments.filter(adj => adj.employeeCedula !== selectedEmployee.cedula);
      setAdjustments(updatedAdjustments);
      toast({ title: 'Ajustes Limpiados', description: `Se eliminaron los ajustes para ${selectedEmployee.name}.` });
  };

  const goToNextEmployee = () => {
    setCurrentEmployeeIndex(prev => (prev + 1) % activeEmployees.length);
  }

  const goToPreviousEmployee = () => {
    setCurrentEmployeeIndex(prev => (prev - 1 + activeEmployees.length) % activeEmployees.length);
  }

  if (activeEmployees.length === 0) {
      return (
        <div className="flex items-center justify-center h-full border rounded-lg bg-muted/30 py-20">
            <p className="text-muted-foreground">No hay empleados activos para procesar la nómina.</p>
        </div>
      )
  }

  const currentConceptList = type === 'assignment' ? activePaymentConcepts : activeDeductionConcepts;

  return (
    <div className="grid grid-cols-1 gap-8 lg:grid-cols-3">
      <div className="lg:col-span-1 print-hide">
        <Card>
          <CardHeader>
            <CardTitle>Revisión de Nómina</CardTitle>
            <CardDescription>
              Navegue entre los recibos de los empleados para revisar y realizar ajustes individuales.
            </CardDescription>
          </CardHeader>
          <CardContent className="space-y-4">
             <div className="flex items-center justify-between gap-2 p-4 border rounded-lg">
                <Button variant="outline" size="icon" onClick={goToPreviousEmployee} disabled={activeEmployees.length <= 1}>
                    <ArrowLeft className="h-4 w-4" />
                </Button>
                <div className="text-center">
                    {selectedEmployee && <p className="font-semibold">{selectedEmployee.name}</p>}
                    <p className="text-sm text-muted-foreground">Empleado {currentEmployeeIndex + 1} de {activeEmployees.length}</p>
                </div>
                <Button variant="outline" size="icon" onClick={goToNextEmployee} disabled={activeEmployees.length <= 1}>
                    <ArrowRight className="h-4 w-4" />
                </Button>
            </div>
            
             <Separator />
             <div className="space-y-2">
                <Dialog open={isDialogOpen} onOpenChange={setIsDialogOpen}>
                <DialogTrigger asChild>
                    <Button variant="outline" className="w-full">
                    <PlusCircle className="mr-2 h-4 w-4" />
                    Ajuste Individual
                    </Button>
                </DialogTrigger>
                <DialogContent className="sm:max-w-[425px]">
                    <DialogHeader>
                    <DialogTitle>Agregar Ajuste Individual</DialogTitle>
                    <DialogDescription>
                        Añada una asignación o deducción puntual para {selectedEmployee?.name}. Los cambios se reflejarán en tiempo real.
                    </DialogDescription>
                    </DialogHeader>
                    <div className="grid gap-4 py-4">
                        <div className="space-y-2">
                            <Label>Tipo de Ajuste</Label>
                            <RadioGroup defaultValue="assignment" value={type} onValueChange={(value: 'assignment' | 'deduction') => setType(value)}>
                            <div className="flex items-center space-x-2">
                                <RadioGroupItem value="assignment" id="r-assignment" />
                                <Label htmlFor="r-assignment">Asignación</Label>
                            </div>
                            <div className="flex items-center space-x-2">
                                <RadioGroupItem value="deduction" id="r-deduction" />
                                <Label htmlFor="r-deduction">Deducción</Label>
                            </div>
                            </RadioGroup>
                        </div>

                    <div className="space-y-2">
                        <Label htmlFor="adj-concept">Concepto</Label>
                        <Select value={concept} onValueChange={setConcept}>
                            <SelectTrigger id="adj-concept">
                                <SelectValue placeholder="Seleccione un concepto" />
                            </SelectTrigger>
                            <SelectContent>
                                <SelectItem value="manual">Otro (descripción manual)</SelectItem>
                                <Separator className="my-1"/>
                                {currentConceptList.map(c => (
                                    <SelectItem key={c.id} value={c.id}>{c.name}</SelectItem>
                                ))}
                            </SelectContent>
                        </Select>
                    </div>
                    
                    {concept === 'manual' && (
                        <div className="space-y-2">
                            <Label htmlFor="adj-description">Descripción Manual</Label>
                            <Input id="adj-description" value={description} onChange={(e) => setDescription(e.target.value)} placeholder="Ej. Préstamo personal" />
                        </div>
                    )}
                        <div className="space-y-2">
                            <Label>Moneda del Monto</Label>
                            <RadioGroup value={adjustmentCurrency} onValueChange={(value: 'VES' | 'USD') => setAdjustmentCurrency(value)}>
                                <div className="flex items-center space-x-2">
                                    <RadioGroupItem value="VES" id="r-ves-adj" />
                                    <Label htmlFor="r-ves-adj">Bolívares (VES)</Label>
                                </div>
                                <div className="flex items-center space-x-2">
                                    <RadioGroupItem value="USD" id="r-usd-adj" />
                                    <Label htmlFor="r-usd-adj">Dólares (USD)</Label>
                                </div>
                            </RadioGroup>
                        </div>

                    <div className="space-y-2">
                        <Label htmlFor="adj-amount">Monto ({adjustmentCurrency})</Label>
                        <Input id="adj-amount" type="number" value={amount} onChange={(e) => setAmount(e.target.value)} placeholder="Ej. 500.00" />
                    </div>
                    </div>
                    <DialogFooter>
                    <Button type="button" onClick={handleAddAdjustment}>Agregar o Actualizar Ajuste</Button>
                    </DialogFooter>
                </DialogContent>
                </Dialog>
                <div className="grid grid-cols-2 gap-2">
                    <Button variant="secondary" onClick={handleSaveAdjustments}>
                        <Save className="mr-2 h-4 w-4" /> Guardar Ajustes
                    </Button>
                    <AlertDialog>
                      <AlertDialogTrigger asChild>
                          <Button variant="destructive" className="w-full">
                            <Trash2 className="mr-2 h-4 w-4" /> Limpiar Ajustes
                          </Button>
                      </AlertDialogTrigger>
                      <AlertDialogContent>
                          <AlertDialogHeader>
                            <AlertDialogTitle>¿Está seguro?</AlertDialogTitle>
                            <AlertDialogDescription>
                              Esta acción eliminará todos los ajustes individuales para {selectedEmployee?.name}. No se puede deshacer.
                            </AlertDialogDescription>
                          </AlertDialogHeader>
                          <AlertDialogFooter>
                              <AlertDialogCancel>Cancelar</AlertDialogCancel>
                              <AlertDialogAction onClick={handleClearAdjustments}>Sí, Limpiar</AlertDialogAction>
                          </AlertDialogFooter>
                      </AlertDialogContent>
                    </AlertDialog>
                </div>
             </div>
          </CardContent>
          <CardFooter>
            <Button className="w-full" onClick={handleProcessPayroll} disabled={isProcessing}>
              <Share2 className="mr-2 h-4 w-4" /> {isProcessing ? 'Procesando...' : 'Compartir / Exportar Nómina'}
            </Button>
          </CardFooter>
        </Card>
      </div>

      <div className="lg:col-span-2">
       {selectedEmployee ? (
            <Receipt 
                employee={selectedEmployee} 
                adjustments={adjustments} 
                companyName={companyName} 
                companyRif={companyRif} 
                companyAddress={companyAddress} 
                bcvRate={bcvRate} 
                activePaymentConcepts={activePaymentConcepts}
                activeDeductionConcepts={activeDeductionConcepts}
                payrollPeriod={payrollPeriod}
                onDownloadPdf={handleDownloadIndividualReceipt}
            />
        ) : (
             <div className="flex items-center justify-center h-full border rounded-lg bg-muted/30">
                 <p className="text-muted-foreground">Seleccione un empleado para ver su recibo.</p>
             </div>
        )}
      </div>
    </div>
  );
}

// 2. Severance Pay Tab (Prestaciones Sociales)
function SeverancePayTab() {
  const [employees, setEmployees] = useState<Employee[]>([]);
  const [selectedEmployeeId, setSelectedEmployeeId] = useState('');
  const [terminationDate, setTerminationDate] = useState(format(new Date(), 'yyyy-MM-dd'));
  const [calculation, setCalculation] = useState<any>(null);
  const [bcvRate, setBcvRate] = useState(defaultBcvRate);
  const [activePaymentConcepts, setActivePaymentConcepts] = useState<Concept[]>([]);
  
  const [companyName, setCompanyName] = useState(defaultCompanyName);
  const [companyRif, setCompanyRif] = useState(defaultCompanyRif);
  const [companyAddress, setCompanyAddress] = useState(defaultCompanyAddress);
  const { toast } = useToast();
  
  const employee = employees.find(e => e.cedula === selectedEmployeeId);
  
  useEffect(() => {
    const loadData = () => {
        const savedEmployees = localStorage.getItem('employees');
        const savedBcvRate = localStorage.getItem('bcvRate');
        const savedPaymentConcepts = localStorage.getItem('paymentConcepts');

        setEmployees(savedEmployees ? JSON.parse(savedEmployees) : initialEmployees);
        setCompanyName(localStorage.getItem('companyName') || defaultCompanyName);
        setCompanyRif(localStorage.getItem('companyRif') || defaultCompanyRif);
        setCompanyAddress(localStorage.getItem('companyAddress') || defaultCompanyAddress);
        setBcvRate(savedBcvRate ? parseFloat(savedBcvRate) : defaultBcvRate);
        setActivePaymentConcepts(savedPaymentConcepts ? JSON.parse(savedPaymentConcepts).filter((c: Concept) => c.active) : initialPaymentConcepts.filter(c => c.active));
    };
    loadData();
    window.addEventListener('storage', loadData);
    return () => {
        window.removeEventListener('storage', loadData);
    };
  }, []);

  const handleCalculate = () => {
    if (!employee) return;

    const seniority = calculateSeniority(employee.hireDate, terminationDate);
    const { dailyIntegralSalary, normalMonthlySalary } = calculateIntegralSalary(employee, activePaymentConcepts, bcvRate);
    
    // Garantía de Prestaciones (Art. 142 LOTTT, literal a y c)
    // Se acredita con 15 días por trimestre, calculado con base en el último salario.
    const guaranteeDays = Math.floor(seniority.totalMonths / 3) * 15;
    const guaranteeAmount = guaranteeDays * (normalMonthlySalary / 30); // Based on last normal salary
    
    // Días adicionales (Art. 142 LOTTT, literal b)
    // Dos días por cada año, acumulativos, hasta un máximo de 30 días.
    const additionalDaysPerYear = 2;
    let totalAdditionalDays = (seniority.years - 1) * additionalDaysPerYear;
    if (totalAdditionalDays < 0) totalAdditionalDays = 0;
    if (totalAdditionalDays > 30) totalAdditionalDays = 30; // Capped at 30
    const additionalAmount = totalAdditionalDays * dailyIntegralSalary;

    // Total
    const totalSeverance = guaranteeAmount + additionalAmount;

    setCalculation({
        employee,
        terminationDate,
        seniority,
        dailyIntegralSalary,
        normalMonthlySalary,
        guaranteeDays,
        guaranteeAmount,
        totalAdditionalDays,
        additionalAmount,
        totalSeverance
    });
  }
  
  const shareOrExport = async (format: 'pdf' | 'excel') => {
    if (!calculation) return;

    const fileName = `liquidacion_${calculation.employee.cedula}`;
    let file: File;
    let title = 'Reporte de Liquidación';

    if (format === 'pdf') {
      const doc = new jsPDF({ orientation: 'p', unit: 'pt', format: 'letter' });
      doc.text(title, 14, 15);
       (doc as any).autoTable({
            body: [
                ['Empleado', calculation.employee.name],
                ['Cédula', calculation.employee.cedula],
                ['Fecha de Ingreso', format(parseISO(calculation.employee.hireDate), 'dd/MM/yyyy')],
                ['Fecha de Egreso', format(parseISO(calculation.terminationDate), 'dd/MM/yyyy')],
                ['Antigüedad', `${calculation.seniority.years} años y ${calculation.seniority.months} meses`],
                ['Último Salario Normal Mensual', formatCurrency(calculation.normalMonthlySalary, 'VES')],
                ['Último Salario Integral Diario', formatCurrency(calculation.dailyIntegralSalary, 'VES')],
                ['--- CONCEPTOS ---', ''],
                ['Garantía de Prestaciones (Art. 142)', formatCurrency(calculation.guaranteeAmount, 'VES')],
                ['Días Adicionales (Art. 142)', formatCurrency(calculation.additionalAmount, 'VES')],
                ['--- TOTAL ---', ''],
                ['Total a Pagar', formatCurrency(calculation.totalSeverance, 'VES')],
            ],
            startY: 20
        });
      const blob = doc.output('blob');
      file = new File([blob], `${fileName}.pdf`, { type: 'application/pdf' });
    } else {
      const worksheet = XLSX.utils.json_to_sheet([
        {
          "Empleado": calculation.employee.name,
          "Cédula": calculation.employee.cedula,
          "Total a Pagar (VES)": calculation.totalSeverance
        }
      ]);
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, "Liquidacion");
      const excelBuffer = XLSX.write(workbook, { bookType: 'xlsx', type: 'array' });
      const blob = new Blob([excelBuffer], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });
      file = new File([blob], `${fileName}.xlsx`, { type: blob.type });
    }
    
    if (navigator.canShare && navigator.canShare({ files: [file] })) {
        try {
            await navigator.share({ files: [file], title: title });
            toast({ title: 'Éxito', description: 'Reporte compartido.' });
        } catch (error) {
            if ((error as Error).name !== 'AbortError') {
                saveAs(file);
                toast({ variant: 'destructive', title: 'Error al Compartir', description: 'El reporte se ha descargado.' });
            }
        }
    } else {
        saveAs(file);
        toast({ title: 'Reporte Descargado' });
    }
  };


  return (
    <div className="grid grid-cols-1 gap-8 lg:grid-cols-3">
      <div className="lg:col-span-1">
        <Card>
          <CardHeader>
            <CardTitle>Cálculo de Prestaciones Sociales</CardTitle>
            <CardDescription>
              Calcule la liquidación para un empleado al finalizar la relación laboral.
            </CardDescription>
          </CardHeader>
          <CardContent className="space-y-4">
            <div className="space-y-2">
              <Label htmlFor="employee-select-severance">Seleccionar Empleado</Label>
              <Select value={selectedEmployeeId} onValueChange={setSelectedEmployeeId}>
                <SelectTrigger id="employee-select-severance">
                  <SelectValue placeholder="Busque y seleccione..." />
                </SelectTrigger>
                <SelectContent>
                  {employees.map(e => <SelectItem key={e.cedula} value={e.cedula}>{e.name}</SelectItem>)}
                </SelectContent>
              </Select>
            </div>
            <div className="space-y-2">
                <Label htmlFor="termination-date">Fecha de Egreso</Label>
                <Input id="termination-date" type="date" value={terminationDate} onChange={(e) => setTerminationDate(e.target.value)} />
            </div>
          </CardContent>
          <CardFooter>
            <Button onClick={handleCalculate} disabled={!selectedEmployeeId}>
                <CalculatorIcon className="mr-2 h-4 w-4" /> Calcular Prestaciones
            </Button>
          </CardFooter>
        </Card>
      </div>
       <div className="lg:col-span-2">
        {calculation ? (
             <Card>
                <CardHeader>
                     <div className="flex items-start justify-between">
                        <div>
                            <ReportHeader companyName={companyName} companyRif={companyRif} companyAddress={companyAddress} />
                            <CardTitle className='mt-4'>Reporte de Liquidación</CardTitle>
                            <CardDescription>
                                Empleado: {calculation.employee.name} | C.I: {calculation.employee.cedula}
                            </CardDescription>
                        </div>
                        <DropdownMenu>
                            <DropdownMenuTrigger asChild><Button variant="outline" size="icon"><MoreVertical className="h-4 w-4"/></Button></DropdownMenuTrigger>
                            <DropdownMenuContent>
                                <DropdownMenuItem onClick={() => shareOrExport('pdf')}><Share2 className="mr-2 h-4 w-4"/> Compartir/Exportar PDF</DropdownMenuItem>
                                <DropdownMenuItem onClick={() => shareOrExport('excel')}><Share2 className="mr-2 h-4 w-4"/> Compartir/Exportar Excel</DropdownMenuItem>
                            </DropdownMenuContent>
                        </DropdownMenu>
                    </div>
                </CardHeader>
                <CardContent className="space-y-4 text-sm">
                    <div className="p-4 bg-muted/50 rounded-lg">
                        <h3 className="font-semibold mb-2">Datos del Cálculo</h3>
                        <div className="grid grid-cols-2 gap-2">
                            <p><strong>Fecha de Ingreso:</strong> {format(parseISO(calculation.employee.hireDate), 'dd/MM/yyyy')}</p>
                            <p><strong>Fecha de Egreso:</strong> {format(parseISO(calculation.terminationDate), 'dd/MM/yyyy')}</p>
                            <p><strong>Antigüedad:</strong> {calculation.seniority.years} años y {calculation.seniority.months} meses</p>
                            <p><strong>Salario Normal Mensual:</strong> {formatCurrency(calculation.normalMonthlySalary, 'VES')}</p>
                            <p><strong>Salario Integral Diario:</strong> {formatCurrency(calculation.dailyIntegralSalary, 'VES')}</p>
                        </div>
                    </div>
                    <div>
                        <h3 className="font-semibold text-primary mb-2">Conceptos a Pagar</h3>
                         <div className="space-y-2">
                            <div className="flex justify-between">
                                <p>Garantía de Prestaciones (Art. 142 LOTTT)</p>
                                <p className='font-medium'>{formatCurrency(calculation.guaranteeAmount, 'VES')}</p>
                            </div>
                            <p className="text-xs text-muted-foreground ml-4">({calculation.guaranteeDays} días acreditados a razón del último Salario Normal)</p>
                             <div className="flex justify-between">
                                <p>Días Adicionales de Prestaciones (Art. 142 LOTTT)</p>
                                <p className='font-medium'>{formatCurrency(calculation.additionalAmount, 'VES')}</p>
                            </div>
                            <p className="text-xs text-muted-foreground ml-4">({calculation.totalAdditionalDays} días adicionales a razón de Salario Integral Diario)</p>
                         </div>
                    </div>
                    <Separator />
                     <div className="flex justify-between font-bold text-lg">
                        <p>Total a Pagar por Liquidación</p>
                        <div className='text-right'>
                            <p className="text-primary">{formatCurrency(calculation.totalSeverance, 'VES')}</p>
                            <p className="text-base font-semibold text-muted-foreground">{formatCurrency(calculation.totalSeverance / bcvRate, 'USD')}</p>
                        </div>
                    </div>
                    <ReportFooter employee={calculation.employee} companyName={companyName} companyRif={companyRif} />
                </CardContent>
            </Card>
        ) : (
            <div className="flex items-center justify-center h-full border rounded-lg bg-muted/30">
                <p className="text-muted-foreground">Los resultados del cálculo aparecerán aquí.</p>
            </div>
        )}
      </div>
    </div>
  )
}

// 3. Vacation Pay Tab
function VacationPayTab() {
  const [employees, setEmployees] = useState<Employee[]>([]);
  const [selectedEmployeeId, setSelectedEmployeeId] = useState('');
  const [calculation, setCalculation] = useState<any>(null);
  const [bcvRate, setBcvRate] = useState(defaultBcvRate);
  const [activePaymentConcepts, setActivePaymentConcepts] = useState<Concept[]>([]);
  const { toast } = useToast();

  const [companyName, setCompanyName] = useState(defaultCompanyName);
  const [companyRif, setCompanyRif] = useState(defaultCompanyRif);
  const [companyAddress, setCompanyAddress] = useState(defaultCompanyAddress);

  const employee = employees.find(e => e.cedula === selectedEmployeeId);
  const today = format(new Date(), 'yyyy-MM-dd');
  
  useEffect(() => {
    const loadData = () => {
        const savedEmployees = localStorage.getItem('employees');
        const savedBcvRate = localStorage.getItem('bcvRate');
        const savedPaymentConcepts = localStorage.getItem('paymentConcepts');
        setEmployees(savedEmployees ? JSON.parse(savedEmployees) : initialEmployees);
        setCompanyName(localStorage.getItem('companyName') || defaultCompanyName);
        setCompanyRif(localStorage.getItem('companyRif') || defaultCompanyRif);
        setCompanyAddress(localStorage.getItem('companyAddress') || defaultCompanyAddress);
        setBcvRate(savedBcvRate ? parseFloat(savedBcvRate) : defaultBcvRate);
        setActivePaymentConcepts(savedPaymentConcepts ? JSON.parse(savedPaymentConcepts).filter((c: Concept) => c.active) : initialPaymentConcepts.filter(c => c.active));
    };
    loadData();
    window.addEventListener('storage', loadData);
    return () => {
        window.removeEventListener('storage', loadData);
    };
  }, []);

  const handleCalculate = () => {
    if (!employee) return;
    
    const seniority = calculateSeniority(employee.hireDate, today);
    const { dailyNormalSalary } = calculateIntegralSalary(employee, activePaymentConcepts, bcvRate);
    const vacationDays = 15 + (seniority.years > 1 ? seniority.years - 1 : 0); // 15 days for first year, +1 for each subsequent year

    const vacationPayment = vacationDays * dailyNormalSalary;
    
    // Vacation bonus is also calculated based on normal salary
    const vacationBonusPayment = vacationDays * dailyNormalSalary; 
    
    const totalPayment = vacationPayment + vacationBonusPayment;

    setCalculation({
        employee,
        seniority,
        vacationDays,
        dailyNormalSalary,
        vacationPayment,
        vacationBonusPayment,
        totalPayment
    });
  }

  const shareOrExport = async (format: 'pdf' | 'excel') => {
    if (!calculation) return;

    const fileName = `vacaciones_${calculation.employee.cedula}`;
    let file: File;
    let title = 'Reporte de Pago de Vacaciones';

    if (format === 'pdf') {
        const doc = new jsPDF({ orientation: 'p', unit: 'pt', format: 'letter' });
        doc.text(title, 14, 15);
         (doc as any).autoTable({
            body: [
                ['Empleado', calculation.employee.name],
                ['Cédula', calculation.employee.cedula],
                ['Antigüedad', `${calculation.seniority.years} años y ${calculation.seniority.months} meses`],
                ['Días de Vacaciones', calculation.vacationDays],
                ['--- CONCEPTOS ---', ''],
                ['Pago por Días de Vacaciones', formatCurrency(calculation.vacationPayment, 'VES')],
                ['Bono Vacacional', formatCurrency(calculation.vacationBonusPayment, 'VES')],
                ['--- TOTAL ---', ''],
                ['Total a Pagar', formatCurrency(calculation.totalPayment, 'VES')],
            ],
            startY: 20
        });
        const blob = doc.output('blob');
        file = new File([blob], `${fileName}.pdf`, { type: 'application/pdf' });
    } else {
        const worksheet = XLSX.utils.json_to_sheet([
          {
            "Empleado": calculation.employee.name,
            "Cédula": calculation.employee.cedula,
            "Total a Pagar (VES)": calculation.totalPayment,
          }
        ]);
        const workbook = XLSX.utils.book_new();
        XLSX.utils.book_append_sheet(workbook, worksheet, "Vacaciones");
        const excelBuffer = XLSX.write(workbook, { bookType: 'xlsx', type: 'array' });
        const blob = new Blob([excelBuffer], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });
        file = new File([blob], `${fileName}.xlsx`, { type: blob.type });
    }
    
    if (navigator.canShare && navigator.canShare({ files: [file] })) {
        try {
            await navigator.share({ files: [file], title: title });
            toast({ title: 'Éxito', description: 'Reporte compartido.' });
        } catch (error) {
             if ((error as Error).name !== 'AbortError') {
                saveAs(file);
                toast({ variant: 'destructive', title: 'Error al Compartir', description: 'El reporte se ha descargado.' });
            }
        }
    } else {
        saveAs(file);
        toast({ title: 'Reporte Descargado' });
    }
  };

  return (
    <div className="grid grid-cols-1 gap-8 lg:grid-cols-3">
        <div className="lg:col-span-1">
            <Card>
                <CardHeader>
                    <CardTitle>Cálculo de Vacaciones</CardTitle>
                    <CardDescription>
                    Calcule el pago de vacaciones y bono vacacional para un empleado.
                    </CardDescription>
                </CardHeader>
                <CardContent className="space-y-4">
                    <div className="space-y-2">
                    <Label htmlFor="employee-select-vacation">Seleccionar Empleado</Label>
                    <Select value={selectedEmployeeId} onValueChange={setSelectedEmployeeId}>
                        <SelectTrigger id="employee-select-vacation">
                        <SelectValue placeholder="Busque y seleccione..." />
                        </SelectTrigger>
                        <SelectContent>
                        {employees.map(e => <SelectItem key={e.cedula} value={e.cedula} disabled={e.status !== 'Activo'}>{e.name}</SelectItem>)}
                        </SelectContent>
                    </Select>
                    </div>
                </CardContent>
                <CardFooter>
                    <Button onClick={handleCalculate} disabled={!selectedEmployeeId}><CalculatorIcon className="mr-2 h-4 w-4"/>Calcular Vacaciones</Button>
                </CardFooter>
            </Card>
        </div>
        <div className="lg:col-span-2">
        {calculation ? (
             <Card>
                <CardHeader>
                     <div className="flex items-start justify-between">
                        <div>
                            <ReportHeader companyName={companyName} companyRif={companyRif} companyAddress={companyAddress} />
                            <CardTitle className='mt-4'>Reporte de Pago de Vacaciones</CardTitle>
                            <CardDescription>
                                Empleado: {calculation.employee.name} | C.I: {calculation.employee.cedula}
                            </CardDescription>
                        </div>
                        <DropdownMenu>
                            <DropdownMenuTrigger asChild><Button variant="outline" size="icon"><MoreVertical className="h-4 w-4"/></Button></DropdownMenuTrigger>
                            <DropdownMenuContent>
                               <DropdownMenuItem onClick={() => shareOrExport('pdf')}><Share2 className="mr-2 h-4 w-4"/> Compartir/Exportar PDF</DropdownMenuItem>
                               <DropdownMenuItem onClick={() => shareOrExport('excel')}><Share2 className="mr-2 h-4 w-4"/> Compartir/Exportar Excel</DropdownMenuItem>
                            </DropdownMenuContent>
                        </DropdownMenu>
                    </div>
                </CardHeader>
                <CardContent className="space-y-4 text-sm">
                    <div className="p-4 bg-muted/50 rounded-lg">
                        <h3 className="font-semibold mb-2">Datos del Cálculo</h3>
                        <div className="grid grid-cols-2 gap-2">
                             <p><strong>Antigüedad:</strong> {calculation.seniority.years} años y {calculation.seniority.months} meses</p>
                            <p><strong>Días de Vacaciones (Art. 190 LOTTT):</strong> {calculation.vacationDays} días</p>
                            <p><strong>Salario Normal Diario:</strong> {formatCurrency(calculation.dailyNormalSalary, 'VES')}</p>
                        </div>
                    </div>
                    <div>
                        <h3 className="font-semibold text-primary mb-2">Conceptos a Pagar</h3>
                         <div className="space-y-2">
                            <div className="flex justify-between">
                                <p>Pago por Días de Vacaciones Disfrutadas</p>
                                <p className='font-medium'>{formatCurrency(calculation.vacationPayment, 'VES')}</p>
                            </div>
                             <div className="flex justify-between">
                                <p>Bono Vacacional (Art. 192 LOTTT)</p>
                                <p className='font-medium'>{formatCurrency(calculation.vacationBonusPayment, 'VES')}</p>
                            </div>
                         </div>
                    </div>
                    <Separator />
                     <div className="flex justify-between font-bold text-lg">
                        <p>Total a Pagar por Vacaciones</p>
                         <div className='text-right'>
                            <p className="text-primary">{formatCurrency(calculation.totalPayment, 'VES')}</p>
                            <p className="text-base font-semibold text-muted-foreground">{formatCurrency(calculation.totalPayment / bcvRate, 'USD')}</p>
                        </div>
                    </div>
                    <ReportFooter employee={calculation.employee} companyName={companyName} companyRif={companyRif} />
                </CardContent>
            </Card>
        ) : (
            <div className="flex items-center justify-center h-full border rounded-lg bg-muted/30">
                <p className="text-muted-foreground">Los resultados del cálculo aparecerán aquí.</p>
            </div>
        )}
        </div>
    </div>
  )
}

// 4. Utilities/Bonus Tab (Aguinaldos)
function UtilitiesPayTab() {
  const [employees, setEmployees] = useState<Employee[]>([]);
  const [daysToPay, setDaysToPay] = useState(30);
  const [calculation, setCalculation] = useState<any>(null);
  const [bcvRate, setBcvRate] = useState(defaultBcvRate);
  const [activePaymentConcepts, setActivePaymentConcepts] = useState<Concept[]>([]);
  const { toast } = useToast();
  
  const [companyName, setCompanyName] = useState(defaultCompanyName);
  const [companyRif, setCompanyRif] = useState(defaultCompanyRif);
  const [companyAddress, setCompanyAddress] = useState(defaultCompanyAddress);

  useEffect(() => {
    const loadData = () => {
        const savedEmployees = localStorage.getItem('employees');
        const savedBcvRate = localStorage.getItem('bcvRate');
        const savedPaymentConcepts = localStorage.getItem('paymentConcepts');
        setEmployees(savedEmployees ? JSON.parse(savedEmployees) : initialEmployees);
        setCompanyName(localStorage.getItem('companyName') || defaultCompanyName);
        setCompanyRif(localStorage.getItem('companyRif') || defaultCompanyRif);
        setCompanyAddress(localStorage.getItem('companyAddress') || defaultCompanyAddress);
        setBcvRate(savedBcvRate ? parseFloat(savedBcvRate) : defaultBcvRate);
        setActivePaymentConcepts(savedPaymentConcepts ? JSON.parse(savedPaymentConcepts).filter((c: Concept) => c.active) : initialPaymentConcepts.filter(c => c.active));
    };
    loadData();
    window.addEventListener('storage', loadData);
    return () => {
        window.removeEventListener('storage', loadData);
    };
  }, []);


  const handleCalculate = () => {
    const activeEmployees = employees.filter(e => e.status === 'Activo');
    const results = activeEmployees.map(employee => {
        const { dailyIntegralSalary } = calculateIntegralSalary(employee, activePaymentConcepts, bcvRate);
        const amountToPay = dailyIntegralSalary * daysToPay;
        return {
            ...employee,
            amountToPay
        }
    });
    const totalAmount = results.reduce((sum, r) => sum + r.amountToPay, 0);

    setCalculation({ results, totalAmount, daysToPay });
  }

  const shareOrExport = async (format: 'pdf' | 'excel') => {
    if (!calculation) return;

    const fileName = `utilidades_${calculation.daysToPay}_dias`;
    let file: File;
    let title = `Reporte de Utilidades (${calculation.daysToPay} días)`;

    if (format === 'pdf') {
        const doc = new jsPDF({ orientation: 'p', unit: 'pt', format: 'letter' });
        doc.text(title, 14, 15);
        (doc as any).autoTable({
            head: [['Empleado', 'Cédula', 'Monto a Pagar (VES)']],
            body: calculation.results.map((res: any) => [res.name, res.cedula, formatCurrency(res.amountToPay, 'VES')]),
            startY: 20
        });
        const blob = doc.output('blob');
        file = new File([blob], `${fileName}.pdf`, { type: 'application/pdf' });
    } else {
        const worksheet = XLSX.utils.json_to_sheet(calculation.results.map((res: any) => ({
            Empleado: res.name,
            Cedula: res.cedula,
            "Monto a Pagar (VES)": res.amountToPay,
        })));
        const workbook = XLSX.utils.book_new();
        XLSX.utils.book_append_sheet(workbook, worksheet, "Utilidades");
        const excelBuffer = XLSX.write(workbook, { bookType: 'xlsx', type: 'array' });
        const blob = new Blob([excelBuffer], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });
        file = new File([blob], `${fileName}.xlsx`, { type: blob.type });
    }
    
     if (navigator.canShare && navigator.canShare({ files: [file] })) {
        try {
            await navigator.share({ files: [file], title: title });
            toast({ title: 'Éxito', description: 'Reporte compartido.' });
        } catch (error) {
             if ((error as Error).name !== 'AbortError') {
                saveAs(file);
                toast({ variant: 'destructive', title: 'Error al Compartir', description: 'El reporte se ha descargado.' });
            }
        }
    } else {
        saveAs(file);
        toast({ title: 'Reporte Descargado' });
    }
  };

  return (
    <div className="grid grid-cols-1 gap-8 lg:grid-cols-3">
        <div className="lg:col-span-1">
            <Card>
            <CardHeader>
                <CardTitle>Cálculo de Utilidades / Aguinaldos</CardTitle>
                <CardDescription>
                Realice el cálculo del pago de utilidades o aguinaldos para fin de año (Art. 131 LOTTT).
                </CardDescription>
            </CardHeader>
            <CardContent className="space-y-4">
                <div className="space-y-2">
                    <Label htmlFor="utilities-days">Días de Utilidades a Pagar (Mín. 30)</Label>
                    <Input id="utilities-days" type="number" value={daysToPay} onChange={(e) => setDaysToPay(Number(e.target.value))} placeholder="Ej. 30" />
                </div>
                <p className="text-sm text-muted-foreground">Este cálculo se aplicará a todos los empleados activos.</p>
            </CardContent>
            <CardFooter>
                <Button onClick={handleCalculate}><CalculatorIcon className="mr-2 h-4 w-4"/>Calcular Utilidades para Todos</Button>
            </CardFooter>
            </Card>
        </div>
        <div className="lg:col-span-2">
            {calculation ? (
                 <Card>
                <CardHeader>
                     <div className="flex items-start justify-between">
                        <div>
                            <ReportHeader companyName={companyName} companyRif={companyRif} companyAddress={companyAddress} />
                            <CardTitle className='mt-4'>Reporte de Utilidades ({calculation.daysToPay} días)</CardTitle>
                            <CardDescription>
                                Total a Pagar: {formatCurrency(calculation.totalAmount, 'VES')}
                            </CardDescription>
                        </div>
                        <DropdownMenu>
                            <DropdownMenuTrigger asChild><Button variant="outline" size="icon"><MoreVertical className="h-4 w-4"/></Button></DropdownMenuTrigger>
                            <DropdownMenuContent>
                                <DropdownMenuItem onClick={() => shareOrExport('pdf')}><Share2 className="mr-2 h-4 w-4"/> Compartir/Exportar PDF</DropdownMenuItem>
                                <DropdownMenuItem onClick={() => shareOrExport('excel')}><Share2 className="mr-2 h-4 w-4"/> Compartir/Exportar Excel</DropdownMenuItem>
                            </DropdownMenuContent>
                        </DropdownMenu>
                    </div>
                </CardHeader>
                <CardContent>
                    <table className="w-full text-sm">
                        <thead>
                            <tr className="border-b">
                                <th className="text-left font-semibold p-2">Empleado</th>
                                <th className="text-right font-semibold p-2">Monto a Pagar (VES)</th>
                            </tr>
                        </thead>
                        <tbody>
                        {calculation.results.map((res: any) => (
                            <tr key={res.cedula} className="border-b">
                                <td className="p-2">{res.name}</td>
                                <td className="text-right p-2">{formatCurrency(res.amountToPay, 'VES')}</td>
                            </tr>
                        ))}
                        </tbody>
                    </table>
                     <ReportFooter employee={null} companyName={companyName} companyRif={companyRif} />
                </CardContent>
            </Card>

            ) : (
                <div className="flex items-center justify-center h-full border rounded-lg bg-muted/30">
                    <p className="text-muted-foreground">Los resultados del cálculo aparecerán aquí.</p>
                </div>
            )}
        </div>
    </div>
  )
}


export default function CalculationsPage() {
  return (
    <div className="space-y-6">
       <Tabs defaultValue="payroll">
            <TabsList className="grid w-full grid-cols-4">
                <TabsTrigger value="payroll">
                    <User className="mr-2 h-4 w-4" /> Nómina Quincenal
                </TabsTrigger>
                <TabsTrigger value="severance">
                    <Briefcase className="mr-2 h-4 w-4" /> Prestaciones Sociales
                </TabsTrigger>
                <TabsTrigger value="vacation">
                    <Plane className="mr-2 h-4 w-4" /> Vacaciones
                </TabsTrigger>
                <TabsTrigger value="utilities">
                    <Gift className="mr-2 h-4 w-4" /> Utilidades / Aguinaldos
                </TabsTrigger>
            </TabsList>
            <TabsContent value="payroll" className="mt-6">
                <PayrollCalculationTab />
            </TabsContent>
            <TabsContent value="severance" className="mt-6">
                <SeverancePayTab />
            </TabsContent>
            <TabsContent value="vacation" className="mt-6">
                <VacationPayTab />
            </TabsContent>
            <TabsContent value="utilities" className="mt-6">
                <UtilitiesPayTab />
            </TabsContent>
        </Tabs>
    </div>
  );
}
// =================================================================================
// FILE: src/app/dashboard/compliance/page.tsx
import { ComplianceChecker } from '@/components/compliance-checker';

export default function CompliancePage() {
  return (
    <div className="space-y-6">
      <div className="flex items-start justify-between">
        <div>
          <h2 className="text-2xl font-bold tracking-tight">
            Cumplimiento Legal y Herramientas IA
          </h2>
          <p className="text-muted-foreground">
            Monitoree el cumplimiento y utilice herramientas de IA para el análisis de contratos.
          </p>
        </div>
      </div>
      <ComplianceChecker />
    </div>
  );
}
// =================================================================================
// FILE: src/app/dashboard/employees/page.tsx
'use client';
import { useState, useEffect } from 'react';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { employees as initialEmployees, Employee, bcvRate as defaultBcvRate, employeeStatuses, EmployeeStatus } from '@/lib/placeholder-data';
import { MoreHorizontal, PlusCircle, FileDown, Share2 } from 'lucide-react';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuTrigger,
  DropdownMenuSeparator
} from '@/components/ui/dropdown-menu';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@/components/ui/dialog';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Badge } from '@/components/ui/badge';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { useToast } from '@/hooks/use-toast';
import * as XLSX from 'xlsx';
import { saveAs } from 'file-saver';
import jsPDF from 'jspdf';
import 'jspdf-autotable';

function EmployeeFormDialog({
  onSave,
  trigger,
  employee
}: {
  onSave: (employee: Employee) => void,
  trigger: React.ReactNode,
  employee?: Employee | null
}) {
  const [isOpen, setIsOpen] = useState(false);
  const [name, setName] = useState('');
  const [cedula, setCedula] = useState('');
  const [position, setPosition] = useState('');
  const [hireDate, setHireDate] = useState('');
  const [baseSalaryBs, setBaseSalaryBs] = useState(0);
  const [status, setStatus] = useState<EmployeeStatus>('Activo');
  
  const { toast } = useToast();

  useEffect(() => {
    if (employee) {
      setName(employee.name);
      setCedula(employee.cedula);
      setPosition(employee.position);
      setHireDate(employee.hireDate);
      setBaseSalaryBs(employee.baseSalaryBs);
      setStatus(employee.status);
    } else {
       setName('');
       setCedula('');
       setPosition('');
       setHireDate('');
       setBaseSalaryBs(0);
       setStatus('Activo');
    }
  }, [employee, isOpen]);


  const handleSubmit = () => {
    if (!name || !cedula || !position || !hireDate || baseSalaryBs <= 0) {
      toast({
        variant: 'destructive',
        title: 'Error de validación',
        description: 'Por favor, complete todos los campos.',
      });
      return;
    }

    const employeeData: Employee = {
      name,
      cedula,
      position,
      hireDate,
      baseSalaryBs,
      status,
      department: employee?.department || 'N/A', // Preserve department if it exists
    };
    onSave(employeeData);
    setIsOpen(false);
  };
  

  return (
    <Dialog open={isOpen} onOpenChange={setIsOpen}>
      <DialogTrigger asChild>{trigger}</DialogTrigger>
      <DialogContent className="sm:max-w-[425px]">
        <DialogHeader>
          <DialogTitle>{employee ? 'Editar Empleado' : 'Agregar Nuevo Empleado'}</DialogTitle>
          <DialogDescription>
            {employee ? 'Actualice los detalles del empleado.' : 'Complete los detalles del nuevo empleado.'}
          </DialogDescription>
        </DialogHeader>
        <div className="grid gap-4 py-4">
          <div className="grid grid-cols-4 items-center gap-4">
            <Label htmlFor="name" className="text-right">Nombre</Label>
            <Input id="name" className="col-span-3" placeholder="Ej. Juan Pérez" value={name} onChange={(e) => setName(e.target.value)} />
          </div>
          <div className="grid grid-cols-4 items-center gap-4">
            <Label htmlFor="cedula" className="text-right">Cédula</Label>
            <Input id="cedula" className="col-span-3" placeholder="Ej. V-12345678" value={cedula} onChange={(e) => setCedula(e.target.value)} disabled={!!employee} />
          </div>
          <div className="grid grid-cols-4 items-center gap-4">
            <Label htmlFor="position" className="text-right">Cargo</Label>
            <Input id="position" className="col-span-3" placeholder="Ej. Analista de Datos" value={position} onChange={(e) => setPosition(e.target.value)} />
          </div>
          <div className="grid grid-cols-4 items-center gap-4">
            <Label htmlFor="hireDate" className="text-right">Fecha Ingreso</Label>
            <Input id="hireDate" type="date" className="col-span-3" value={hireDate} onChange={(e) => setHireDate(e.target.value)} />
          </div>
          <div className="grid grid-cols-4 items-center gap-4">
            <Label htmlFor="salary" className="text-right">Salario Base (Bs.)</Label>
            <Input id="salary" type="number" className="col-span-3" placeholder="Ej. 130" value={baseSalaryBs} onChange={(e) => setBaseSalaryBs(Number(e.target.value))} />
          </div>
          <div className="grid grid-cols-4 items-center gap-4">
            <Label htmlFor="status" className="text-right">Estatus</Label>
            <Select value={status} onValueChange={(value) => setStatus(value as EmployeeStatus)}>
                <SelectTrigger className="col-span-3">
                    <SelectValue placeholder="Seleccione un estatus" />
                </SelectTrigger>
                <SelectContent>
                    {employeeStatuses.map(s => <SelectItem key={s} value={s}>{s}</SelectItem>)}
                </SelectContent>
            </Select>
          </div>
        </div>
        <DialogFooter>
          <Button type="button" onClick={handleSubmit}>Guardar Cambios</Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}

export default function EmployeesPage() {
  const [employees, setEmployees] = useState<Employee[]>([]);
  const [bcvRate, setBcvRate] = useState(defaultBcvRate);
  const { toast } = useToast();
  
  const saveEmployees = (updatedEmployees: Employee[]) => {
    localStorage.setItem('employees', JSON.stringify(updatedEmployees));
    setEmployees(updatedEmployees);
    window.dispatchEvent(new Event('storage'));
  };

  useEffect(() => {
    const loadData = () => {
        const savedEmployees = localStorage.getItem('employees');
        const savedBcvRate = localStorage.getItem('bcvRate');
        setEmployees(savedEmployees ? JSON.parse(savedEmployees) : initialEmployees);
        setBcvRate(savedBcvRate ? parseFloat(savedBcvRate) : defaultBcvRate);
    };

    loadData();
    window.addEventListener('storage', loadData);

    return () => {
        window.removeEventListener('storage', loadData);
    };
  }, []);

  const addEmployee = (employee: Employee) => {
    if (employees.find(e => e.cedula === employee.cedula)) {
      toast({ variant: 'destructive', title: 'Error', description: 'Un empleado con esa cédula ya existe.' });
      return;
    }
    const updatedEmployees = [...employees, employee];
    saveEmployees(updatedEmployees);
    toast({ title: 'Éxito', description: 'Empleado agregado correctamente.' });
  };
  
  const editEmployee = (updatedEmployee: Employee) => {
    const updatedEmployees = employees.map(e => e.cedula === updatedEmployee.cedula ? updatedEmployee : e);
    saveEmployees(updatedEmployees);
    toast({ title: 'Éxito', description: 'Empleado actualizado correctamente.' });
  };

  const deleteEmployee = (cedula: string) => {
    const updatedEmployees = employees.filter(e => e.cedula !== cedula);
    saveEmployees(updatedEmployees);
    toast({ title: 'Éxito', description: 'Empleado eliminado.' });
  }

  const getStatusVariant = (status: EmployeeStatus) => {
    switch (status) {
      case 'Activo': return 'default';
      case 'Reposo': return 'secondary';
      case 'Vacaciones': return 'outline';
      case 'Retirado': return 'destructive';
      default: return 'default';
    }
  }

  const formatCurrency = (amount: number, currency: 'USD' | 'VES') => {
    return new Intl.NumberFormat('es-VE', {
      style: 'currency',
      currency: currency,
      minimumFractionDigits: 2,
    }).format(amount);
  };
  
  const shareOrExport = async (format: 'pdf' | 'excel') => {
    let file: File;
    const fileName = "lista_empleados";
    const title = "Lista de Empleados";

    if (format === 'pdf') {
        const doc = new jsPDF();
        doc.text(title, 14, 15);
        (doc as any).autoTable({
            head: [['Nombre', 'Cédula', 'Cargo', 'Estatus', 'Salario Base']],
            body: employees.map(emp => [
                emp.name,
                emp.cedula,
                emp.position,
                emp.status,
                formatCurrency(emp.baseSalaryBs, 'VES')
            ]),
            startY: 20
        });
        const blob = doc.output('blob');
        file = new File([blob], `${fileName}.pdf`, { type: 'application/pdf' });
    } else { // excel
        const worksheet = XLSX.utils.json_to_sheet(employees);
        const workbook = XLSX.utils.book_new();
        XLSX.utils.book_append_sheet(workbook, worksheet, "Empleados");
        const excelBuffer = XLSX.write(workbook, { bookType: 'xlsx', type: 'array' });
        const blob = new Blob([excelBuffer], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });
        file = new File([blob], `${fileName}.xlsx`, { type: blob.type });
    }

    if (navigator.canShare && navigator.canShare({ files: [file] })) {
        try {
            await navigator.share({ files: [file], title: title });
            toast({ title: 'Éxito', description: 'Lista de empleados compartida.' });
        } catch (error) {
            if ((error as Error).name !== 'AbortError') {
                saveAs(file);
                toast({ variant: 'destructive', title: 'Error al Compartir', description: 'El archivo se ha descargado.' });
            }
        }
    } else {
        saveAs(file);
        toast({ title: 'Archivo Descargado', description: 'La lista de empleados ha sido descargada.' });
    }
};

  return (
    <Card>
      <CardHeader className="flex flex-row items-center justify-between">
        <div>
          <CardTitle>Gestión de Empleados</CardTitle>
          <CardDescription>
            Los cambios que realice aquí se guardan y sincronizan en tiempo real con toda la aplicación.
          </CardDescription>
        </div>
        <div className="flex items-center gap-2">
            <DropdownMenu>
              <DropdownMenuTrigger asChild>
                <Button variant="outline" className="gap-1">
                  <Share2 className="h-3.5 w-3.5" />
                  <span className="sr-only sm:not-sr-only sm:whitespace-nowrap">Compartir</span>
                </Button>
              </DropdownMenuTrigger>
              <DropdownMenuContent align="end">
                <DropdownMenuLabel>Opciones de Reporte</DropdownMenuLabel>
                <DropdownMenuSeparator />
                <DropdownMenuItem onClick={() => shareOrExport('excel')}>Exportar a Excel (.xlsx)</DropdownMenuItem>
                <DropdownMenuItem onClick={() => shareOrExport('pdf')}>Exportar a PDF (.pdf)</DropdownMenuItem>
              </DropdownMenuContent>
            </DropdownMenu>
            <EmployeeFormDialog 
              onSave={addEmployee}
              trigger={
                <Button size="sm" className="gap-1">
                  <PlusCircle className="h-3.5 w-3.5" />
                  <span className="sr-only sm:not-sr-only sm:whitespace-nowrap">
                    Agregar Empleado
                  </span>
                </Button>
              }
            />
        </div>
      </CardHeader>
      <CardContent>
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Nombre</TableHead>
              <TableHead className="hidden md:table-cell">Cédula</TableHead>
              <TableHead>Cargo</TableHead>
              <TableHead>Estatus</TableHead>
              <TableHead className="text-right">Salario Base Mensual</TableHead>
              <TableHead>
                <span className="sr-only">Acciones</span>
              </TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {employees.map((employee: Employee) => (
              <TableRow key={employee.cedula}>
                <TableCell className="font-medium">{employee.name}</TableCell>
                <TableCell className="hidden md:table-cell">
                  {employee.cedula}
                </TableCell>
                <TableCell>{employee.position}</TableCell>
                <TableCell>
                  <Badge variant={getStatusVariant(employee.status)}>{employee.status}</Badge>
                </TableCell>
                <TableCell className="text-right">
                  <div className='font-medium'>{formatCurrency(employee.baseSalaryBs, 'VES')}</div>
                  <div className='text-xs text-muted-foreground'>{formatCurrency(employee.baseSalaryBs / bcvRate, 'USD')}</div>
                </TableCell>
                <TableCell>
                  <DropdownMenu>
                    <DropdownMenuTrigger asChild>
                      <Button aria-haspopup="true" size="icon" variant="ghost">
                        <MoreHorizontal className="h-4 w-4" />
                        <span className="sr-only">Toggle menu</span>
                      </Button>
                    </DropdownMenuTrigger>
                    <DropdownMenuContent align="end">
                      <DropdownMenuLabel>Acciones</DropdownMenuLabel>
                       <EmployeeFormDialog
                          employee={employee}
                          onSave={editEmployee}
                          trigger={<DropdownMenuItem onSelect={(e) => e.preventDefault()}>Editar</DropdownMenuItem>}
                        />
                      <DropdownMenuItem>Ver Perfil</DropdownMenuItem>
                      <DropdownMenuSeparator />
                      <DropdownMenuItem className="text-destructive" onClick={() => deleteEmployee(employee.cedula)}>
                        Eliminar
                      </DropdownMenuItem>
                    </DropdownMenuContent>
                  </DropdownMenu>
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </CardContent>
    </Card>
  );
}
// =================================================================================
// FILE: src/app/dashboard/layout.tsx
'use client';

import * as React from 'react';
import Link from 'next/link';
import { usePathname } from 'next/navigation';
import {
  SidebarProvider,
  Sidebar,
  SidebarHeader,
  SidebarContent,
  SidebarMenu,
  SidebarMenuItem,
  SidebarMenuButton,
  SidebarFooter,
  SidebarTrigger,
  SidebarInset,
} from '@/components/ui/sidebar';
import {
  Users,
  Home,
  FileText,
  Calculator,
  ShieldCheck,
  Settings,
  Wallet,
  LogOut,
  ChevronDown,
  History,
  UploadCloud,
  FileClock,
} from 'lucide-react';
import { AppLogo } from '@/components/icons';
import { Button } from '@/components/ui/button';
import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { SheetTitle } from '@/components/ui/sheet';

const menuItems = [
  {
    href: '/dashboard',
    label: 'Inicio',
    icon: Home,
  },
  {
    href: '/dashboard/employees',
    label: 'Empleados',
    icon: Users,
  },
  {
    href: '/dashboard/payroll-concepts',
    label: 'Conceptos de Nómina',
    icon: Wallet,
  },
  {
    href: '/dashboard/load-payroll',
    label: 'Cargar Nómina',
    icon: UploadCloud,
  },
  {
    href: '/dashboard/calculations',
    label: 'Cálculos y Reportes',
    icon: Calculator,
  },
  {
    href: '/dashboard/compliance',
    label: 'Cumplimiento Legal',
    icon: ShieldCheck,
  },
  {
    href: '/dashboard/payroll-history',
    label: 'Historial de Nómina',
    icon: History,
  },
];

const secondaryMenuItems = [
    {
    href: '/dashboard/access-report',
    label: 'Reporte de Accesos',
    icon: FileClock,
  },
  {
    href: '/dashboard/settings',
    label: 'Ajustes',
    icon: Settings,
  },
];

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const pathname = usePathname();
  const allMenuItems = [...menuItems, ...secondaryMenuItems];
  const [appName, setAppName] = React.useState('Nómina Global VE');
  const [appLogo, setAppLogo] = React.useState<React.ReactNode>(<AppLogo className="h-8 w-8 text-sidebar-primary" />);

  React.useEffect(() => {
    const loadSettings = () => {
        const savedAppName = localStorage.getItem('appName');
        const savedAppLogoUrl = localStorage.getItem('appLogoUrl');
        if (savedAppName) {
            setAppName(savedAppName);
        }
        if (savedAppLogoUrl) {
            setAppLogo(<img src={savedAppLogoUrl} alt="App Logo" className="h-8 w-8" />);
        } else {
            setAppLogo(<AppLogo className="h-8 w-8 text-sidebar-primary" />);
        }
    };
    
    loadSettings();
    window.addEventListener('storage', loadSettings);

    return () => {
        window.removeEventListener('storage', loadSettings);
    };
  }, []);

  return (
    <SidebarProvider>
      <Sidebar>
        <SidebarHeader className="border-b border-sidebar-border">
          <div className="flex items-center gap-2 p-2">
            {appLogo}
            <span className="text-lg font-semibold">{appName}</span>
          </div>
        </SidebarHeader>
        <SidebarContent>
          <SidebarMenu>
            {menuItems.map((item) => (
              <SidebarMenuItem key={item.href}>
                <SidebarMenuButton
                    asChild
                    isActive={pathname.startsWith(item.href) && (item.href !== '/dashboard' || pathname === '/dashboard')}
                    tooltip={item.label}
                  >
                  <Link href={item.href}>
                    <item.icon />
                    <span>{item.label}</span>
                  </Link>
                </SidebarMenuButton>
              </SidebarMenuItem>
            ))}
          </SidebarMenu>
        </SidebarContent>
        <SidebarFooter>
           <SidebarMenu className="p-2">
            {secondaryMenuItems.map((item) => (
              <SidebarMenuItem key={item.href}>
                  <SidebarMenuButton
                    asChild
                    isActive={pathname.startsWith(item.href)}
                    tooltip={item.label}
                  >
                    <Link href={item.href}>
                      <item.icon />
                      <span>{item.label}</span>
                    </Link>
                  </SidebarMenuButton>
              </SidebarMenuItem>
            ))}
          </SidebarMenu>
          <DropdownMenu>
            <DropdownMenuTrigger asChild>
              <Button
                variant="ghost"
                className="flex h-auto w-full items-center justify-start gap-2 p-2"
              >
                <Avatar className="h-8 w-8">
                  <AvatarImage
                    src="https://placehold.co/40x40"
                    alt="Admin User"
                    data-ai-hint="user avatar"
                  />
                  <AvatarFallback>AD</AvatarFallback>
                </Avatar>
                <div className="hidden text-left group-data-[collapsible=icon]:hidden">
                  <p className="font-medium">Admin</p>
                  <p className="text-xs text-muted-foreground">
                    admin@example.com
                  </p>
                </div>
                <ChevronDown className="ml-auto hidden h-4 w-4 group-data-[collapsible=icon]:hidden" />
              </Button>
            </DropdownMenuTrigger>
            <DropdownMenuContent side="right" align="start" className="w-56">
              <DropdownMenuLabel>Mi Cuenta</DropdownMenuLabel>
              <DropdownMenuSeparator />
              <DropdownMenuItem asChild>
                <Link href="/dashboard/settings">
                  <Settings className="mr-2 h-4 w-4" />
                  <span>Configuración</span>
                </Link>
              </DropdownMenuItem>
              <DropdownMenuSeparator />
              <DropdownMenuItem asChild>
                <Link href="/">
                  <LogOut className="mr-2 h-4 w-4" />
                  <span>Cerrar Sesión</span>
                </Link>
              </DropdownMenuItem>
            </DropdownMenuContent>
          </DropdownMenu>
        </SidebarFooter>
      </Sidebar>
      <SidebarInset>
        <header className="flex h-14 items-center gap-4 border-b bg-card px-6">
          <SidebarTrigger className="md:hidden" />
          <div className="flex-1">
            <h1 className="text-lg font-semibold md:text-xl">
              {allMenuItems.find((item) => pathname.startsWith(item.href) && (item.href !== '/dashboard' || pathname === '/dashboard'))?.label || 'Dashboard'}
            </h1>
          </div>
        </header>
        <main className="flex-1 overflow-auto p-4 md:p-6">{children}</main>
      </SidebarInset>
    </SidebarProvider>
  );
}
// =================================================================================
// FILE: src/app/dashboard/load-payroll/page.tsx
'use client';
import { useState, useEffect } from 'react';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { employees as initialEmployees, Employee, bcvRate as defaultBcvRate, EmployeeStatus, PendingPayroll, paymentConcepts as initialPaymentConcepts, deductionConcepts as initialDeductionConcepts, Concept } from '@/lib/placeholder-data';
import { Badge } from '@/components/ui/badge';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { useToast } from '@/hooks/use-toast';
import { Banknote, CheckCircle } from 'lucide-react';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { format } from 'date-fns';
import { es } from 'date-fns/locale';
import { getReceiptData } from '@/components/receipt';

function getStatusVariant(status: EmployeeStatus) {
    switch (status) {
      case 'Activo': return 'default';
      case 'Reposo': return 'secondary';
      case 'Vacaciones': return 'outline';
      case 'Retirado': return 'destructive';
      default: return 'default';
    }
}

export default function LoadPayrollPage() {
  const { toast } = useToast();
  const [employees, setEmployees] = useState<Employee[]>([]);
  const [bcvRate, setBcvRate] = useState(defaultBcvRate.toString());
  const [activePaymentConcepts, setActivePaymentConcepts] = useState<Concept[]>([]);
  const [activeDeductionConcepts, setActiveDeductionConcepts] = useState<Concept[]>([]);

  const [month, setMonth] = useState(format(new Date(), 'MMMM', { locale: es }));
  const [year, setYear] = useState(new Date().getFullYear().toString());
  const [fortnight, setFortnight] = useState<'first' | 'second'>('first');

  useEffect(() => {
    const loadData = () => {
        const savedEmployees = localStorage.getItem('employees');
        const savedBcvRate = localStorage.getItem('bcvRate');
        const savedPaymentConcepts = localStorage.getItem('paymentConcepts');
        const savedDeductionConcepts = localStorage.getItem('deductionConcepts');
        
        const savedYear = localStorage.getItem('payrollYear');
        const savedMonth = localStorage.getItem('payrollMonth');
        const savedFortnight = localStorage.getItem('payrollFortnight');

        setEmployees(savedEmployees ? JSON.parse(savedEmployees) : initialEmployees);
        setBcvRate(savedBcvRate || defaultBcvRate.toString());
        
        const allPaymentConcepts = savedPaymentConcepts ? JSON.parse(savedPaymentConcepts) : initialPaymentConcepts;
        setActivePaymentConcepts(allPaymentConcepts.filter((c: Concept) => c.active));
        const allDeductionConcepts = savedDeductionConcepts ? JSON.parse(savedDeductionConcepts) : initialDeductionConcepts;
        setActiveDeductionConcepts(allDeductionConcepts.filter((c: Concept) => c.active));

        if (savedYear) setYear(savedYear);
        if (savedMonth) setMonth(savedMonth);
        if (savedFortnight) setFortnight(savedFortnight as 'first' | 'second');

    };

    loadData();

    window.addEventListener('storage', loadData);

    return () => {
        window.removeEventListener('storage', loadData);
    };
  }, []);
  
  const activeEmployees = employees.filter(e => e.status === 'Activo');


  const handleLoadPayroll = () => {
    const numericBcvRate = parseFloat(bcvRate);
    if (!month || !year || !bcvRate || isNaN(numericBcvRate)) {
        toast({
            variant: 'destructive',
            title: 'Error de validación',
            description: 'Debe seleccionar período, año e introducir una tasa BCV válida.',
        });
        return;
    }
    
    // Save settings to localStorage so other pages can use them
    localStorage.setItem('bcvRate', bcvRate);
    localStorage.setItem('payrollYear', year);
    localStorage.setItem('payrollMonth', month);
    localStorage.setItem('payrollFortnight', fortnight);

    const fortnightText = fortnight === 'first' ? 'Primera' : 'Segunda';
    const periodText = `${fortnightText} Quincena ${month} ${year}`;
    
    // Calculate total amount in real-time
    const totalPayrollAmount = activeEmployees.reduce((total, employee) => {
        const { netToPayBs } = getReceiptData({
            employee,
            adjustments: [], // No individual adjustments when loading bulk payroll
            bcvRate: numericBcvRate,
            activePaymentConcepts,
            activeDeductionConcepts,
            payrollPeriod: { year, month, fortnight }
        });
        return total + netToPayBs;
    }, 0);

    const newPendingPayroll: PendingPayroll = {
        id: `NOM-${year}-${month.substring(0,3).toUpperCase()}-Q${fortnight === 'first' ? 1 : 2}`,
        period: periodText,
        totalAmountBs: totalPayrollAmount,
        employees: activeEmployees.length,
        status: 'pending'
    };

    const savedPending = localStorage.getItem('pendingPayroll');
    const pending = savedPending ? JSON.parse(savedPending) : [];
    
    if (pending.find((p: PendingPayroll) => p.id === newPendingPayroll.id)) {
        toast({
            variant: 'destructive',
            title: 'Nómina Duplicada',
            description: `La nómina para este período ya ha sido cargada. Puede verla en "Historial de Nómina".`,
        });
        return;
    }

    pending.push(newPendingPayroll);
    localStorage.setItem('pendingPayroll', JSON.stringify(pending));


    window.dispatchEvent(new Event('storage')); // Notify other tabs/components

    toast({
        title: 'Nómina Cargada',
        description: `Se ha cargado la nómina para ${month} (${fortnight === 'first' ? '1ra' : '2da'} Quincena) del ${year} con ${activeEmployees.length} empleados activos y Tasa BCV de ${bcvRate}.`,
    });
  }

  return (
    <div className="space-y-6">
        <Card>
            <CardHeader>
                <CardTitle>Cargar Nueva Nómina</CardTitle>
                <CardDescription>
                    Seleccione el período, introduzca la tasa de cambio del día y verifique los empleados activos que se incluirán.
                </CardDescription>
            </CardHeader>
            <CardContent className="space-y-4">
                 <div className="grid grid-cols-1 md:grid-cols-4 gap-4 items-end">
                    <div className="space-y-2">
                        <Label htmlFor="year">Año</Label>
                        <Input id="year" type="number" value={year} onChange={(e) => setYear(e.target.value)} placeholder="Ej. 2024" />
                    </div>
                    <div className="space-y-2">
                        <Label htmlFor="period">Mes</Label>
                        <Select value={month} onValueChange={setMonth}>
                            <SelectTrigger id="period">
                                <SelectValue placeholder="Seleccione un mes" />
                            </SelectTrigger>
                            <SelectContent>
                                <SelectItem value="Enero">Enero</SelectItem>
                                <SelectItem value="Febrero">Febrero</SelectItem>
                                <SelectItem value="Marzo">Marzo</SelectItem>
                                <SelectItem value="Abril">Abril</SelectItem>
                                <SelectItem value="Mayo">Mayo</SelectItem>
                                <SelectItem value="Junio">Junio</SelectItem>
                                <SelectItem value="Julio">Julio</SelectItem>
                                <SelectItem value="Agosto">Agosto</SelectItem>
                                <SelectItem value="Septiembre">Septiembre</SelectItem>
                                <SelectItem value="Octubre">Octubre</SelectItem>
                                <SelectItem value="Noviembre">Noviembre</SelectItem>
                                <SelectItem value="Diciembre">Diciembre</SelectItem>
                            </SelectContent>
                        </Select>
                    </div>
                    <div className="space-y-2">
                        <Label htmlFor="fortnight">Quincena</Label>
                        <Select value={fortnight} onValueChange={(v) => setFortnight(v as 'first' | 'second')}>
                            <SelectTrigger id="fortnight">
                                <SelectValue />
                            </SelectTrigger>
                            <SelectContent>
                                <SelectItem value="first">Primera Quincena</SelectItem>
                                <SelectItem value="second">Segunda Quincena</SelectItem>
                            </SelectContent>
                        </Select>
                    </div>
                     <div className="space-y-2">
                        <Label htmlFor="bcv-rate" className="flex items-center">
                            <Banknote className="mr-2 h-4 w-4 text-muted-foreground" />
                            Tasa BCV del Día
                        </Label>
                        <Input id="bcv-rate" type="number" value={bcvRate} onChange={(e) => setBcvRate(e.target.value)} placeholder="Ej. 36.42" />
                    </div>
                 </div>
                 <div className="flex justify-end mt-4">
                     <Button onClick={handleLoadPayroll} className="w-full md:w-auto">
                        <CheckCircle className="mr-2 h-4 w-4" />
                        Cargar Nómina
                    </Button>
                 </div>
            </CardContent>
        </Card>
        
        <Card>
            <CardHeader>
                <CardTitle>Empleados a Incluir en Nómina</CardTitle>
                <CardDescription>
                Esta es la lista de empleados con estatus "Activo" que se incluirán en el próximo cálculo.
                </CardDescription>
            </CardHeader>
            <CardContent>
                <Table>
                <TableHeader>
                    <TableRow>
                    <TableHead>Nombre</TableHead>
                    <TableHead className="hidden md:table-cell">Cédula de Identidad</TableHead>
                    <TableHead>Cargo</TableHead>
                    <TableHead>Estatus</TableHead>
                    </TableRow>
                </TableHeader>
                <TableBody>
                    {activeEmployees.map((employee: Employee) => (
                    <TableRow key={employee.cedula}>
                        <TableCell className="font-medium">{employee.name}</TableCell>
                        <TableCell className="hidden md:table-cell">{employee.cedula}</TableCell>
                        <TableCell>{employee.position}</TableCell>
                        <TableCell>
                            <Badge variant={getStatusVariant(employee.status)}>{employee.status}</Badge>
                        </TableCell>
                    </TableRow>
                    ))}
                    {activeEmployees.length === 0 && (
                        <TableRow>
                            <TableCell colSpan={4} className="text-center text-muted-foreground">
                                No hay empleados activos para cargar en la nómina.
                            </TableCell>
                        </TableRow>
                    )}
                </TableBody>
                </Table>
            </CardContent>
        </Card>
    </div>
  );
}
// =================================================================================
// FILE: src/app/dashboard/page.tsx
'use client';

import { useState, useEffect } from 'react';
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from '@/components/ui/card';
import { payrollChartData, employees as initialEmployees, bcvRate as defaultBcvRate, Employee, PayrollHistory } from '@/lib/placeholder-data';
import { Users, FileWarning, CalendarCheck, Banknote } from 'lucide-react';
import {
  Bar,
  BarChart,
  CartesianGrid,
  ResponsiveContainer,
  Tooltip,
  XAxis,
  YAxis,
} from 'recharts';
import { ChartConfig, ChartContainer } from '@/components/ui/chart';
import { format, addDays, lastDayOfMonth, parse } from 'date-fns';
import { es } from 'date-fns/locale';


const chartConfig = {
  total: {
    label: 'Total',
    color: 'hsl(var(--chart-2))',
  },
} satisfies ChartConfig;

export default function DashboardPage() {
    const [employees, setEmployees] = useState<Employee[]>([]);
    const [bcvRate, setBcvRate] = useState(defaultBcvRate);
    const [nextPayrollDate, setNextPayrollDate] = useState('Calculando...');
    const [totalMonthlyPayrollUsd, setTotalMonthlyPayrollUsd] = useState(0);

    useEffect(() => {
        const loadData = () => {
            const savedEmployees = localStorage.getItem('employees');
            const savedBcvRate = localStorage.getItem('bcvRate');
            const currentEmployees = savedEmployees ? JSON.parse(savedEmployees) : initialEmployees;
            const currentBcvRate = savedBcvRate ? parseFloat(savedBcvRate) : defaultBcvRate;
            
            setEmployees(currentEmployees);
            setBcvRate(currentBcvRate);
            
            const savedPaidPayroll = localStorage.getItem('paidPayroll');
            const paidPayroll: PayrollHistory[] = savedPaidPayroll ? JSON.parse(savedPaidPayroll) : [];
            
            if (paidPayroll.length > 0) {
                const lastPayroll = paidPayroll[0]; // The list is sorted, so the first one is the latest
                const period = lastPayroll.period.toLowerCase();
                setTotalMonthlyPayrollUsd(lastPayroll.totalAmountBs / currentBcvRate);
                
                const monthMatch = period.match(/(enero|febrero|marzo|abril|mayo|junio|julio|agosto|septiembre|octubre|noviembre|diciembre)/);
                const yearMatch = period.match(/\d{4}/);
                
                if(monthMatch && yearMatch) {
                    const monthName = monthMatch[0];
                    const year = parseInt(yearMatch[0]);
                    
                    const dateString = `01 ${monthName} ${year}`;
                    const lastPeriodDate = parse(dateString, 'dd MMMM yyyy', new Date(), { locale: es });

                    let nextDate: Date;
                    if (period.includes('primera')) {
                        // Next is the end of the same month
                        nextDate = lastDayOfMonth(lastPeriodDate);
                    } else { // segunda quincena
                        // Next is the 15th of the next month
                        const nextMonth = new Date(year, lastPeriodDate.getMonth() + 1, 15);
                        nextDate = nextMonth;
                    }
                     setNextPayrollDate(format(nextDate, "dd 'de' MMMM, yyyy", { locale: es }));
                }

            } else {
                 setNextPayrollDate("Pendiente de primer pago");
                 setTotalMonthlyPayrollUsd(0);
            }
        };
        
        loadData();
        window.addEventListener('storage', loadData);
        return () => {
            window.removeEventListener('storage', loadData);
        };
    }, []);

    const activeEmployeesCount = employees.filter(e => e.status === 'Activo').length;

  return (
    <div className="space-y-6">
      <div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-4">
        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">
              Total de Empleados
            </CardTitle>
            <Users className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">{employees.length} ({activeEmployeesCount} activos)</div>
            <p className="text-xs text-muted-foreground">
              Sincronizado en tiempo real
            </p>
          </CardContent>
        </Card>
        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">
              Alertas de Cumplimiento
            </CardTitle>
            <FileWarning className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">0</div>
            <p className="text-xs text-muted-foreground">
              No hay alertas pendientes
            </p>
          </CardContent>
        </Card>
        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">
              Próxima Nómina
            </CardTitle>
            <CalendarCheck className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold capitalize">{nextPayrollDate}</div>
            <p className="text-xs text-muted-foreground">Pago quincenal</p>
          </CardContent>
        </Card>
        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">
              Último Costo Nómina
            </CardTitle>
            <Banknote className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">${totalMonthlyPayrollUsd.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2})}</div>
            <p className="text-xs text-muted-foreground">
              Basado en el último pago procesado
            </p>
          </CardContent>
        </Card>
      </div>
      <Card>
        <CardHeader>
          <CardTitle>Costo de Nómina Mensual (USD)</CardTitle>
          <CardDescription>
            Un resumen del costo total de la nómina en los últimos 6 meses.
          </CardDescription>
        </CardHeader>
        <CardContent>
          <ChartContainer config={chartConfig} className="h-[300px] w-full">
            <ResponsiveContainer>
              <BarChart data={payrollChartData} margin={{ top: 20 }}>
                <CartesianGrid vertical={false} />
                <XAxis
                  dataKey="month"
                  tickLine={false}
                  tickMargin={10}
                  axisLine={false}
                />
                <YAxis
                  tickFormatter={(value) => `$${Number(value) / 1000}k`}
                  tickLine={false}
                  axisLine={false}
                  tickMargin={10}
                />
                <Tooltip
                  cursor={false}
                  content={({ active, payload }) => {
                    if (active && payload && payload.length) {
                      return (
                        <div className="rounded-lg border bg-background p-2 shadow-sm">
                          <div className="grid grid-cols-2 gap-2">
                            <div className="flex flex-col">
                              <span className="text-[0.70rem] uppercase text-muted-foreground">
                                Mes
                              </span>
                              <span className="font-bold text-muted-foreground">
                                {payload[0].payload.month}
                              </span>
                            </div>
                            <div className="flex flex-col">
                              <span className="text-[0.70rem] uppercase text-muted-foreground">
                                Total
                              </span>
                              <span className="font-bold">
                                ${payload[0].value?.toLocaleString()}
                              </span>
                            </div>
                          </div>
                        </div>
                      );
                    }

                    return null;
                  }}
                />
                <Bar dataKey="total" fill="var(--color-total)" radius={4} />
              </BarChart>
            </ResponsiveContainer>
          </ChartContainer>
        </CardContent>
      </Card>
    </div>
  );
}
// =================================================================================
// FILE: src/app/dashboard/payroll-concepts/page.tsx
'use client';
import { useState, useEffect } from 'react';
import { Button } from '@/components/ui/button';
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from '@/components/ui/card';
import { paymentConcepts as initialPaymentConcepts, deductionConcepts as initialDeductionConcepts, bcvRate as defaultBcvRate } from '@/lib/placeholder-data';
import { Pencil, Plus, Trash2, Save } from 'lucide-react';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@/components/ui/dialog';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Checkbox } from '@/components/ui/checkbox';
import { useToast } from '@/hooks/use-toast';
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from '@/components/ui/alert-dialog';
import { RadioGroup, RadioGroupItem } from '@/components/ui/radio-group';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';

interface Concept {
    id: string;
    name: string;
    active: boolean;
    isSystemConcept: boolean;
    isEditable?: boolean;
    amountType?: 'VES' | 'USD';
    amountVes?: number;
    amountUsd?: number;
    paymentFrequency?: 'monthly' | 'fortnightly';
}

function EditConceptDialog({ concept, onSave }: { concept: Concept, onSave: (updatedConcept: Concept) => void }) {
  const [isOpen, setIsOpen] = useState(false);
  const [bcvRate, setBcvRate] = useState(defaultBcvRate);
  const [amountType, setAmountType] = useState<'VES' | 'USD'>(concept.amountType || 'VES');
  const [amountVes, setAmountVes] = useState(concept.amountVes || 0);
  const [amountUsd, setAmountUsd] = useState(concept.amountUsd || 0);
  const [paymentFrequency, setPaymentFrequency] = useState<'monthly' | 'fortnightly'>(concept.paymentFrequency || 'monthly');
  const { toast } = useToast();

  useEffect(() => {
    const savedBcvRate = localStorage.getItem('bcvRate');
    setBcvRate(savedBcvRate ? parseFloat(savedBcvRate) : defaultBcvRate);
  }, [isOpen]);

  useEffect(() => {
    setAmountType(concept.amountType || 'VES');
    setAmountVes(concept.amountVes || 0);
    setAmountUsd(concept.amountUsd || 0);
    setPaymentFrequency(concept.paymentFrequency || 'monthly');
  }, [concept, isOpen]);

  const handleAmountChange = (value: string, type: 'VES' | 'USD') => {
    const numericValue = parseFloat(value) || 0;
    if (type === 'VES') {
      setAmountVes(numericValue);
      setAmountUsd(numericValue / bcvRate);
    } else { // USD
      setAmountUsd(numericValue);
      setAmountVes(numericValue * bcvRate);
    }
  };
  
  const handleSave = () => {
    onSave({
      ...concept,
      amountType,
      amountVes,
      amountUsd,
      paymentFrequency,
    });
    toast({ title: "Concepto actualizado", description: `"${concept.name}" ha sido modificado. No olvide guardar los cambios.`});
    setIsOpen(false);
  }

  return (
    <Dialog open={isOpen} onOpenChange={setIsOpen}>
      <DialogTrigger asChild>
        <Button
          variant="ghost"
          size="icon"
          className="h-8 w-8"
          disabled={!concept.isEditable}
          title={concept.isEditable ? `Editar ${concept.name}`: 'Este concepto no es editable globalmente'}
        >
          <Pencil className="h-4 w-4 text-muted-foreground" />
          <span className="sr-only">Editar {concept.name}</span>
        </Button>
      </DialogTrigger>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Editar Monto del Concepto</DialogTitle>
          <DialogDescription>
            Ajuste el monto para &quot;{concept.name}&quot;. Este cambio se aplicará
            globalmente en futuros cálculos. Tasa BCV: {bcvRate}
          </DialogDescription>
        </DialogHeader>
        <div className="grid gap-4 py-4">
          <div className="space-y-2">
            <Label>Frecuencia de Pago</Label>
             <Select value={paymentFrequency} onValueChange={(v) => setPaymentFrequency(v as 'monthly' | 'fortnightly')}>
              <SelectTrigger>
                <SelectValue />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="monthly">Mensual</SelectItem>
                <SelectItem value="fortnightly">Quincenal</SelectItem>
              </SelectContent>
            </Select>
            <p className="text-xs text-muted-foreground">Si es mensual, el monto se dividirá entre dos para el pago quincenal.</p>
          </div>
          <div className="space-y-2">
            <Label>Definir monto en</Label>
            <RadioGroup value={amountType} onValueChange={(v) => setAmountType(v as 'VES' | 'USD')}>
              <div className="flex items-center space-x-2">
                <RadioGroupItem value="VES" id="r-ves" />
                <Label htmlFor="r-ves">Bolívares (VES)</Label>
              </div>
              <div className="flex items-center space-x-2">
                <RadioGroupItem value="USD" id="r-usd" />
                <Label htmlFor="r-usd">Dólares (USD)</Label>
              </div>
            </RadioGroup>
          </div>
          <div className="space-y-2">
            <Label htmlFor="amount-ves">Monto en Bolívares (Bs.)</Label>
            <Input id="amount-ves" type="number" value={amountVes} onChange={(e) => handleAmountChange(e.target.value, 'VES')} disabled={amountType !== 'VES'} />
          </div>
          <div className="space-y-2">
            <Label htmlFor="amount-usd">Monto en Dólares ($)</Label>
            <Input id="amount-usd" type="number" value={amountUsd} onChange={(e) => handleAmountChange(e.target.value, 'USD')} disabled={amountType !== 'USD'} />
          </div>
        </div>
        <DialogFooter>
          <Button onClick={handleSave}>Guardar</Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}

function AddConceptDialog({ onAdd, trigger }: { onAdd: (name: string) => void, trigger: React.ReactNode }) {
    const [isOpen, setIsOpen] = useState(false);
    const [name, setName] = useState('');
    const { toast } = useToast();

    const handleAdd = () => {
        if (!name.trim()) {
            toast({ variant: 'destructive', title: 'Error', description: 'El nombre del concepto no puede estar vacío.'});
            return;
        }
        onAdd(name);
        setName('');
        setIsOpen(false);
    }
    
    return (
        <Dialog open={isOpen} onOpenChange={setIsOpen}>
            <DialogTrigger asChild>{trigger}</DialogTrigger>
            <DialogContent>
                <DialogHeader>
                    <DialogTitle>Agregar Nuevo Concepto</DialogTitle>
                    <DialogDescription>Introduzca el nombre del nuevo concepto.</DialogDescription>
                </DialogHeader>
                 <div className="grid gap-4 py-4">
                    <div className="space-y-2">
                        <Label htmlFor="concept-name">Nombre del Concepto</Label>
                        <Input id="concept-name" value={name} onChange={(e) => setName(e.target.value)} placeholder="Ej. Bono de transporte"/>
                    </div>
                </div>
                <DialogFooter>
                    <Button onClick={handleAdd}>Agregar Concepto</Button>
                </DialogFooter>
            </DialogContent>
        </Dialog>
    )
}

function ConceptList({
  title,
  concepts,
  isPayment,
  onConceptUpdate,
  onConceptAdd,
  onConceptDelete,
}: {
  title: string;
  concepts: Concept[];
  isPayment: boolean;
  onConceptUpdate: (updatedConcept: Concept) => void;
  onConceptAdd: (name: string) => void;
  onConceptDelete: (id: string) => void;
}) {
  return (
    <Card>
      <CardHeader>
        <CardTitle className="flex items-center justify-between">
          <span>{title}</span>
          <AddConceptDialog
            onAdd={onConceptAdd}
            trigger={
              <Button size="sm" variant="outline">
                <Plus className="mr-2 h-4 w-4" /> Agregar
              </Button>
            }
          />
        </CardTitle>
        <CardDescription>
          {isPayment
            ? 'Conceptos que suman al ingreso del empleado.'
            : 'Conceptos que se deducen del ingreso del empleado.'}
        </CardDescription>
      </CardHeader>
      <CardContent>
        <ul className="space-y-2">
          {concepts.map((concept) => (
            <li
              key={concept.id}
              className="flex items-center justify-between rounded-md border p-3"
            >
              <div className="flex items-center gap-3">
                <Checkbox 
                  id={`concept-${concept.id}`} 
                  checked={concept.active}
                  onCheckedChange={(checked) => onConceptUpdate({ ...concept, active: !!checked })}
                />
                <Label htmlFor={`concept-${concept.id}`} className="font-medium">
                  {concept.name}
                </Label>
              </div>
              <div className="flex items-center">
                <EditConceptDialog concept={concept} onSave={onConceptUpdate} />
                <AlertDialog>
                  <AlertDialogTrigger asChild>
                    <Button variant="ghost" size="icon" className="h-8 w-8" disabled={concept.isSystemConcept} title={concept.isSystemConcept ? 'No se puede eliminar un concepto del sistema' : `Eliminar ${concept.name}`}>
                      <Trash2 className="h-4 w-4 text-muted-foreground" />
                      <span className="sr-only">Eliminar {concept.name}</span>
                    </Button>
                  </AlertDialogTrigger>
                  <AlertDialogContent>
                    <AlertDialogHeader>
                      <AlertDialogTitle>¿Está seguro?</AlertDialogTitle>
                      <AlertDialogDescription>
                        Esta acción no se puede deshacer. Esto eliminará permanentemente el concepto &quot;{concept.name}&quot;.
                      </AlertDialogDescription>
                    </AlertDialogHeader>
                    <AlertDialogFooter>
                      <AlertDialogCancel>Cancelar</AlertDialogCancel>
                      <AlertDialogAction onClick={() => onConceptDelete(concept.id)}>
                        Eliminar
                      </AlertDialogAction>
                    </AlertDialogFooter>
                  </AlertDialogContent>
                </AlertDialog>
              </div>
            </li>
          ))}
        </ul>
      </CardContent>
    </Card>
  );
}

export default function PayrollConceptsPage() {
  const { toast } = useToast();
  const [paymentConcepts, setPaymentConcepts] = useState<Concept[]>([]);
  const [deductionConcepts, setDeductionConcepts] = useState<Concept[]>([]);

  useEffect(() => {
    // Load concepts from localStorage or initialize with defaults
    const loadConcepts = () => {
        const savedPaymentConcepts = localStorage.getItem('paymentConcepts');
        const savedDeductionConcepts = localStorage.getItem('deductionConcepts');

        setPaymentConcepts(savedPaymentConcepts ? JSON.parse(savedPaymentConcepts) : initialPaymentConcepts);
        setDeductionConcepts(savedDeductionConcepts ? JSON.parse(savedDeductionConcepts) : initialDeductionConcepts);
    };
    loadConcepts();
  }, []);

  const handleSaveChanges = () => {
    localStorage.setItem('paymentConcepts', JSON.stringify(paymentConcepts));
    localStorage.setItem('deductionConcepts', JSON.stringify(deductionConcepts));
    window.dispatchEvent(new Event('storage'));
    toast({
      title: 'Cambios Guardados',
      description: 'Los conceptos de la nómina han sido actualizados exitosamente.',
    });
  };

  const handleConceptUpdate = (updatedConcept: Concept, type: 'payment' | 'deduction') => {
    const updater = (concepts: Concept[]) => concepts.map(c => c.id === updatedConcept.id ? updatedConcept : c);
    if (type === 'payment') {
        setPaymentConcepts(updater);
    } else {
        setDeductionConcepts(updater);
    }
  };

  const handleConceptAdd = (name: string, type: 'payment' | 'deduction') => {
    const newConcept: Concept = {
        id: `custom-${Date.now()}`,
        name,
        active: true,
        isSystemConcept: false,
        isEditable: true,
        amountType: 'VES',
        amountVes: 0,
        amountUsd: 0,
        paymentFrequency: 'monthly',
    };

    if (type === 'payment') {
        setPaymentConcepts(prev => [...prev, newConcept]);
    } else {
        setDeductionConcepts(prev => [...prev, newConcept]);
    }
    toast({ title: 'Concepto Agregado', description: `"${name}" ha sido añadido. No olvide guardar los cambios.` });
  };

  const handleConceptDelete = (id: string, type: 'payment' | 'deduction') => {
    const updater = (concepts: Concept[]) => concepts.filter(c => c.id !== id);
    if (type === 'payment') {
        setPaymentConcepts(updater);
    } else {
        setDeductionConcepts(updater);
    }
     toast({ title: 'Concepto Eliminado', description: `El concepto ha sido eliminado. No olvide guardar los cambios.`, variant: 'destructive' });
  };

  return (
    <div className="space-y-6">
      <div className="flex items-start justify-between">
        <div>
          <h2 className="text-2xl font-bold tracking-tight">
            Conceptos de Nómina
          </h2>
          <p className="text-muted-foreground">
            Gestione los conceptos de pago y deducciones que se aplican en la
            nómina.
          </p>
        </div>
        <Button onClick={handleSaveChanges}>
            <Save className="mr-2 h-4 w-4" />
            Guardar Cambios
        </Button>
      </div>
      <div className="grid grid-cols-1 gap-6 md:grid-cols-2">
        <ConceptList
          title="Conceptos de Pago"
          concepts={paymentConcepts}
          isPayment={true}
          onConceptUpdate={(concept) => handleConceptUpdate(concept, 'payment')}
          onConceptAdd={(name) => handleConceptAdd(name, 'payment')}
          onConceptDelete={(id) => handleConceptDelete(id, 'payment')}
        />
        <ConceptList
          title="Conceptos de Deducción"
          concepts={deductionConcepts}
          isPayment={false}
          onConceptUpdate={(concept) => handleConceptUpdate(concept, 'deduction')}
          onConceptAdd={(name) => handleConceptAdd(name, 'deduction')}
          onConceptDelete={(id) => handleConceptDelete(id, 'deduction')}
        />
      </div>
    </div>
  );
}
// =================================================================================
// FILE: src/app/dashboard/payroll-history/page.tsx
'use client';
import { useState, useEffect } from 'react';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardDescription, CardHeader, CardTitle, CardFooter } from '@/components/ui/card';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Badge } from '@/components/ui/badge';
import { FileDown, MoreVertical, CheckCircle, Search, User, CreditCard, Share2, Trash2, Download } from 'lucide-react';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
  DropdownMenuSeparator,
  DropdownMenuLabel,
} from '@/components/ui/dropdown-menu';
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from '@/components/ui/alert-dialog';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { employees as initialEmployees, bcvRate as defaultBcvRate, payrollHistory as initialPayrollHistory, Employee, PayrollHistory, PendingPayroll, paymentConcepts as initialPaymentConcepts, deductionConcepts as initialDeductionConcepts, Concept, Adjustment } from '@/lib/placeholder-data';
import { useToast } from '@/hooks/use-toast';
import { Label } from '@/components/ui/label';
import jsPDF from 'jspdf';
import 'jspdf-autotable';
import * as XLSX from 'xlsx';
import { saveAs } from 'file-saver';
import { Receipt, getReceiptData } from '@/components/receipt';
import { format, parse, parseISO } from 'date-fns';
import { es } from 'date-fns/locale';

const formatCurrency = (amount: number, currency: 'USD' | 'VES') => {
  return new Intl.NumberFormat('es-VE', {
    style: 'currency',
    currency: currency,
    minimumFractionDigits: 2,
  }).format(amount);
};

function IndividualHistory() {
  const [employees, setEmployees] = useState<Employee[]>([]);
  const [selectedEmployeeId, setSelectedEmployeeId] = useState('');
  const [bcvRate, setBcvRate] = useState(defaultBcvRate);
  const [paidPayroll, setPaidPayroll] = useState<PayrollHistory[]>([]);
  const [activePaymentConcepts, setActivePaymentConcepts] = useState<Concept[]>([]);
  const [activeDeductionConcepts, setActiveDeductionConcepts] = useState<Concept[]>([]);
  const [companyName, setCompanyName] = useState('');
  const [companyRif, setCompanyRif] = useState('');
  const [companyAddress, setCompanyAddress] = useState('');
  const { toast } = useToast();

  const [selectedPeriodId, setSelectedPeriodId] = useState('');
  
  useEffect(() => {
    const loadData = () => {
        const savedEmployees = localStorage.getItem('employees');
        const savedBcvRate = localStorage.getItem('bcvRate');
        const savedPaidPayroll = localStorage.getItem('paidPayroll');
        const savedPaymentConcepts = localStorage.getItem('paymentConcepts');
        const savedDeductionConcepts = localStorage.getItem('deductionConcepts');
        const savedCompanyName = localStorage.getItem('companyName');
        const savedCompanyRif = localStorage.getItem('companyRif');
        const savedCompanyAddress = localStorage.getItem('companyAddress');
        
        setEmployees(savedEmployees ? JSON.parse(savedEmployees) : initialEmployees);
        setBcvRate(savedBcvRate ? parseFloat(savedBcvRate) : defaultBcvRate);
        setPaidPayroll(savedPaidPayroll ? JSON.parse(savedPaidPayroll) : []);
        setActivePaymentConcepts(savedPaymentConcepts ? JSON.parse(savedPaymentConcepts) : initialPaymentConcepts);
        setActiveDeductionConcepts(savedDeductionConcepts ? JSON.parse(savedDeductionConcepts) : initialDeductionConcepts);
        setCompanyName(savedCompanyName || '');
        setCompanyRif(savedCompanyRif || '');
        setCompanyAddress(savedCompanyAddress || '');
    };
    loadData();
    window.addEventListener('storage', loadData);
    return () => {
        window.removeEventListener('storage', loadData);
    };
  }, []);

  const selectedEmployee = employees.find(e => e.cedula === selectedEmployeeId);
  
  // Reset period selection when employee changes
  useEffect(() => {
    setSelectedPeriodId('');
  }, [selectedEmployeeId]);
  
  const employeePayments = paidPayroll.filter(p => p.payments.some(payment => payment.employeeId === selectedEmployeeId));

  const selectedPayment = paidPayroll.find(p => p.id === selectedPeriodId);
  
  const handleDownloadReceipt = () => {
    if (!selectedPayment || !selectedEmployee) return;

    const doc = new jsPDF({ orientation: 'p', unit: 'pt', format: 'letter' });
    const pageWidth = doc.internal.pageSize.getWidth();
    const pageMargin = 15;

    const payrollPeriod = {
        year: selectedPayment.period.split(' ').pop() || '',
        month: selectedPayment.period.split(' ')[2] || '',
        fortnight: selectedPayment.period.toLowerCase().includes('primera') ? 'first' : 'second' as 'first' | 'second'
    };
    
    // We assume adjustments are stored per payroll, if not, this would be empty
    const adjustments: Adjustment[] = selectedPayment.adjustments || [];

    const receiptData = getReceiptData({
        employee: selectedEmployee,
        adjustments,
        bcvRate,
        activePaymentConcepts,
        activeDeductionConcepts,
        payrollPeriod
    });

    let finalY = 15;

    // Header
    doc.setFontSize(14); doc.setFont('helvetica', 'bold');
    doc.text(companyName, pageWidth / 2, finalY, { align: 'center' }); finalY += 6;
    doc.setFontSize(10); doc.setFont('helvetica', 'normal');
    doc.text(`RIF: ${companyRif}`, pageWidth / 2, finalY, { align: 'center' }); finalY += 4;
    doc.text(companyAddress, pageWidth / 2, finalY, { align: 'center' }); finalY += 10;
    
    // Receipt Info
    doc.setFontSize(12); doc.setFont('helvetica', 'bold');
    doc.text('Recibo de Pago Quincenal', pageMargin, finalY);
    doc.setFontSize(10); doc.setFont('helvetica', 'normal');
    doc.text(`${selectedEmployee.name} | C.I. ${selectedEmployee.cedula}`, pageWidth - pageMargin, finalY, { align: 'right' });
    finalY += 6;
    doc.setTextColor(100);
    doc.text(`${receiptData.periodString} | Tasa BCV: ${formatCurrency(bcvRate, 'VES')}`, pageMargin, finalY); finalY += 6;
    doc.setTextColor(0);
    
    (doc as any).autoTable({
      startY: finalY,
      theme: 'plain',
      styles: { fontSize: 9, cellPadding: { top: 0, bottom: 0 } },
      body: [[`Cargo: ${selectedEmployee.position}`, {content: `Fecha Ingreso: ${format(parseISO(selectedEmployee.hireDate), 'dd/MM/yyyy')}`, styles: { halign: 'right' }}]]
    });
    finalY = (doc as any).lastAutoTable.finalY + 5;

    // Assignments and Deductions
    const assignmentsBody = receiptData.assignmentsList.map(item => [
        { content: `${item.description}${item.note ? `\n${item.note}` : ''}`, styles: { textColor: item.note ? [100, 100, 100] : [0,0,0], cellPadding: {top: 1, bottom: 1} } },
        { content: formatCurrency(item.amountBs, 'VES'), styles: { halign: 'right' } }
    ]);
    const deductionsBody = receiptData.deductionsList.map(item => [
         item.description,
        { content: formatCurrency(-item.amountBs, 'VES'), styles: { halign: 'right' } }
    ]);
    (doc as any).autoTable({
        startY: finalY,
        theme: 'grid',
        headStyles: { fillColor: [240, 240, 240], textColor: [0,0,0], fontStyle: 'bold' },
        columnStyles: { 0: {cellWidth: 80}, 1: { halign: 'right' }, 2: {cellWidth: 80}, 3: { halign: 'right' } },
        head: [['Asignaciones', 'Monto (VES)'], ['Deducciones', 'Monto (VES)']],
        body: assignmentsBody.map((row, i) => [...row, ...(deductionsBody[i] || ['', ''])]),
    });
    finalY = (doc as any).lastAutoTable.finalY + 5;

    // ... rest of PDF generation ...
     doc.save(`recibo_${selectedEmployee.cedula}_${selectedPayment.id}.pdf`);
     toast({title: "Recibo Descargado", description: "El recibo de pago ha sido descargado."});
  };

  return (
    <Card>
      <CardHeader>
        <CardTitle>Historial Individual por Trabajador</CardTitle>
        <CardDescription>
          Seleccione un empleado y un período para ver y descargar su recibo de pago detallado.
        </CardDescription>
      </CardHeader>
      <CardContent className="space-y-4">
        <div className="flex gap-2">
            <Select value={selectedEmployeeId} onValueChange={setSelectedEmployeeId}>
              <SelectTrigger id="employee-search" className="flex-1">
                  <SelectValue placeholder="Seleccione un empleado" />
              </SelectTrigger>
              <SelectContent>
                  {employees.map((e) => (
                  <SelectItem key={e.cedula} value={e.cedula}>
                      {e.name}
                  </SelectItem>
                  ))}
              </SelectContent>
            </Select>
            {selectedEmployee && (
              <Select value={selectedPeriodId} onValueChange={setSelectedPeriodId} disabled={employeePayments.length === 0}>
                  <SelectTrigger id="period-search" className="flex-1">
                      <SelectValue placeholder="Seleccione un período" />
                  </SelectTrigger>
                  <SelectContent>
                      {employeePayments.map((p) => (
                          <SelectItem key={p.id} value={p.id}>
                              {p.period}
                          </SelectItem>
                      ))}
                  </SelectContent>
              </Select>
            )}
        </div>
        
        {selectedEmployee && selectedPeriodId && selectedPayment ? (
            <div className='mt-6'>
                <Receipt 
                    employee={selectedEmployee} 
                    adjustments={selectedPayment.adjustments || []} 
                    companyName={companyName} 
                    companyRif={companyRif} 
                    companyAddress={companyAddress} 
                    bcvRate={bcvRate} 
                    activePaymentConcepts={activePaymentConcepts}
                    activeDeductionConcepts={activeDeductionConcepts}
                    payrollPeriod={{
                        year: selectedPayment.period.split(' ').pop() || '',
                        month: selectedPayment.period.split(' ')[2] || '',
                        fortnight: selectedPayment.period.toLowerCase().includes('primera') ? 'first' : 'second'
                    }}
                />
                <div className="flex justify-end mt-4">
                  <Button onClick={handleDownloadReceipt}>
                    <Download className="mr-2 h-4 w-4"/>
                    Descargar Recibo en PDF
                  </Button>
                </div>
            </div>
        ) : (
          <div className="flex items-center justify-center h-48 border rounded-lg bg-muted/30 mt-6">
              <p className="text-muted-foreground text-center">
                  {selectedEmployee ? 'Seleccione un período para ver el recibo.' : 'Seleccione un empleado para empezar.'}
              </p>
          </div>
        )}
      </CardContent>
    </Card>
  );
}


export default function PayrollHistoryPage() {
    const { toast } = useToast();
    const [employees, setEmployees] = useState<Employee[]>([]);
    const [bcvRate, setBcvRate] = useState(defaultBcvRate);
    const [pendingPayroll, setPendingPayroll] = useState<PendingPayroll[]>([]);
    const [paidPayroll, setPaidPayroll] = useState<PayrollHistory[]>([]);
    const [activePaymentConcepts, setActivePaymentConcepts] = useState<Concept[]>([]);
    const [activeDeductionConcepts, setActiveDeductionConcepts] = useState<Concept[]>([]);
    
    useEffect(() => {
        const loadData = () => {
            const savedEmployees = localStorage.getItem('employees');
            const savedBcvRate = localStorage.getItem('bcvRate');
            setEmployees(savedEmployees ? JSON.parse(savedEmployees) : initialEmployees);
            setBcvRate(savedBcvRate ? parseFloat(savedBcvRate) : defaultBcvRate);

            const savedPending = localStorage.getItem('pendingPayroll');
            setPendingPayroll(savedPending ? JSON.parse(savedPending) : []);
            const savedPaid = localStorage.getItem('paidPayroll');
            setPaidPayroll(savedPaid ? JSON.parse(savedPaid) : []);

            const savedPaymentConcepts = localStorage.getItem('paymentConcepts');
            setActivePaymentConcepts(savedPaymentConcepts ? JSON.parse(savedPaymentConcepts) : initialPaymentConcepts);
            const savedDeductionConcepts = localStorage.getItem('deductionConcepts');
            setActiveDeductionConcepts(savedDeductionConcepts ? JSON.parse(savedDeductionConcepts) : initialDeductionConcepts);
        };
        loadData();
        window.addEventListener('storage', loadData);
        return () => {
            window.removeEventListener('storage', loadData);
        };
    }, []);

    const saveState = (pending: PendingPayroll[], paid: PayrollHistory[]) => {
        setPendingPayroll(pending);
        setPaidPayroll(paid);
        localStorage.setItem('pendingPayroll', JSON.stringify(pending));
        localStorage.setItem('paidPayroll', JSON.stringify(paid));
        window.dispatchEvent(new Event('storage'));
    }

    const handleDeletePendingPayroll = (id: string) => {
        const updatedPending = pendingPayroll.filter(p => p.id !== id);
        saveState(updatedPending, paidPayroll);
        toast({
            title: 'Nómina Eliminada',
            description: `La nómina pendiente ${id} ha sido eliminada.`,
            variant: 'destructive',
        });
    };

    const handleMarkAsPaid = (id: string) => {
        const payrollToMove = pendingPayroll.find(p => p.id === id);
        if (payrollToMove) {
            const updatedPending = pendingPayroll.filter(p => p.id !== id);

            const periodParts = payrollToMove.period.split(' ');
            const payrollPeriod = {
                year: periodParts[3] || '',
                month: periodParts[2] || '',
                fortnight: periodParts[0].toLowerCase() === 'primera' ? 'first' : 'second' as 'first' | 'second'
            };
            
            const activeEmployees = employees.filter(e => e.status === 'Activo');

            const payments = activeEmployees.map(emp => {
                const { totalAssignmentsBs, totalDeductionsBs, netToPayBs } = getReceiptData({
                    employee: emp,
                    adjustments: [], // Assume no individual adjustments for bulk payment
                    bcvRate,
                    activePaymentConcepts,
                    activeDeductionConcepts,
                    payrollPeriod,
                });
                return {
                    periodId: payrollToMove.id,
                    employeeId: emp.cedula,
                    employeeName: emp.name,
                    totalAssignmentsBs,
                    totalDeductionsBs,
                    netToPayBs,
                };
            });

            // Recalculate total amount to ensure accuracy
            const accurateTotalAmount = payments.reduce((sum, p) => sum + p.netToPayBs, 0);
            
            const newPaidEntry: PayrollHistory = {
                id: payrollToMove.id,
                period: payrollToMove.period,
                totalAmountBs: accurateTotalAmount,
                payments: payments,
                adjustments: [], // Store adjustments if any
            };
            const updatedPaid = [newPaidEntry, ...paidPayroll];
            
            saveState(updatedPending, updatedPaid);

            toast({
                title: 'Nómina Pagada',
                description: `La nómina ${id} se ha marcado como pagada.`,
                className: 'bg-green-100 text-green-800'
            });
        }
    };
    
    const shareOrExport = async (payrollData: any[], title: string, fileName: string, format: 'pdf' | 'excel') => {
        let file: File;
        
        if (format === 'pdf') {
            const doc = new jsPDF();
            doc.text(title, 14, 15);
            (doc as any).autoTable({
                head: [['ID Nómina', 'Período', '# Empleados', 'Monto Total (VES)']],
                body: payrollData.map(p => [
                    p.id,
                    p.period,
                    p.employees || p.payments.length,
                    formatCurrency(p.totalAmountBs, 'VES')
                ]),
                startY: 20
            });
            const blob = doc.output('blob');
            file = new File([blob], `${fileName}.pdf`, { type: 'application/pdf' });
        } else {
            const worksheet = XLSX.utils.json_to_sheet(payrollData.map(p => ({
                'ID Nómina': p.id,
                'Período': p.period,
                '# Empleados': p.employees || p.payments.length,
                'Monto Total (VES)': p.totalAmountBs
            })));
            const workbook = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(workbook, worksheet, "Nóminas");
            const excelBuffer = XLSX.write(workbook, { bookType: 'xlsx', type: 'array' });
            const blob = new Blob([excelBuffer], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });
            file = new File([blob], `${fileName}.xlsx`, { type: blob.type });
        }
    
        if (navigator.canShare && navigator.canShare({ files: [file] })) {
            try {
                await navigator.share({ files: [file], title });
                toast({ title: 'Éxito', description: 'Reporte compartido.' });
            } catch (error) {
                if ((error as Error).name !== 'AbortError') {
                    saveAs(file);
                    toast({ variant: 'destructive', title: 'Error al Compartir', description: 'El reporte se ha descargado.' });
                }
            }
        } else {
            saveAs(file);
            toast({ title: 'Reporte Descargado' });
        }
    };


  return (
    <Tabs defaultValue="pending">
      <div className="flex items-center justify-between">
        <div>
          <h2 className="text-2xl font-bold tracking-tight">Historial de Nómina</h2>
          <p className="text-muted-foreground">
            Revise nóminas pendientes, pagadas y el historial de pagos de cada trabajador.
          </p>
        </div>
        <TabsList>
          <TabsTrigger value="pending">Pendientes de Pago</TabsTrigger>
          <TabsTrigger value="paid">Histórico de Pagos</TabsTrigger>
          <TabsTrigger value="individual">Consulta Individual</TabsTrigger>
        </TabsList>
      </div>

      <TabsContent value="pending" className="mt-6">
        <Card>
            <CardHeader className="flex flex-row justify-between items-center">
                <div>
                  <CardTitle>Nóminas Cargadas y No Pagadas</CardTitle>
                  <CardDescription>
                      Estas son las nóminas que han sido calculadas y están pendientes de ser procesadas y pagadas.
                  </CardDescription>
                </div>
                <Button variant="outline" size="sm" onClick={() => shareOrExport(pendingPayroll, 'Nóminas Pendientes', 'nominas_pendientes', 'pdf')}>
                    <Share2 className="mr-2 h-4 w-4"/> Compartir/Exportar
                </Button>
            </CardHeader>
            <CardContent>
                <Table>
                    <TableHeader>
                        <TableRow>
                            <TableHead>ID Nómina</TableHead>
                            <TableHead>Período</TableHead>
                            <TableHead># Empleados</TableHead>
                            <TableHead className="text-right">Monto Total</TableHead>
                            <TableHead className="text-center">Acciones</TableHead>
                        </TableRow>
                    </TableHeader>
                    <TableBody>
                        {pendingPayroll.map(p => (
                            <TableRow key={p.id}>
                                <TableCell><Badge variant="outline">{p.id}</Badge></TableCell>
                                <TableCell>{p.period}</TableCell>
                                <TableCell>{p.employees}</TableCell>
                                <TableCell className="text-right">
                                    <div className="font-medium">{formatCurrency(p.totalAmountBs, 'VES')}</div>
                                    <div className="text-xs text-muted-foreground">{formatCurrency(p.totalAmountBs / bcvRate, 'USD')}</div>
                                </TableCell>
                                <TableCell className="text-center">
                                    <DropdownMenu>
                                    <DropdownMenuTrigger asChild>
                                        <Button size="icon" variant="ghost" className="h-8 w-8">
                                        <MoreVertical className="h-4 w-4" />
                                        </Button>
                                    </DropdownMenuTrigger>
                                    <DropdownMenuContent align="end">
                                        <DropdownMenuLabel>Acciones</DropdownMenuLabel>
                                        <DropdownMenuItem onClick={() => handleMarkAsPaid(p.id)}>
                                            <CheckCircle className="mr-2 h-4 w-4"/> Marcar como Pagada
                                        </DropdownMenuItem>
                                        <DropdownMenuSeparator />
                                        <DropdownMenuItem onClick={() => shareOrExport([p], `Nómina ${p.id}`, `nomina_${p.id}`, 'pdf')}>
                                            <Share2 className="mr-2 h-4 w-4" /> Compartir PDF
                                        </DropdownMenuItem>
                                        <DropdownMenuItem onClick={() => shareOrExport([p], `Nómina ${p.id}`, `nomina_${p.id}`, 'excel')}>
                                            <Share2 className="mr-2 h-4 w-4" /> Compartir Excel
                                        </DropdownMenuItem>
                                        <DropdownMenuSeparator />
                                        <AlertDialog>
                                            <AlertDialogTrigger asChild>
                                                <DropdownMenuItem className="text-destructive" onSelect={(e) => e.preventDefault()}>
                                                    <Trash2 className="mr-2 h-4 w-4"/> Eliminar Nómina
                                                </DropdownMenuItem>
                                            </AlertDialogTrigger>
                                            <AlertDialogContent>
                                                <AlertDialogHeader>
                                                    <AlertDialogTitle>¿Está absolutamente seguro?</AlertDialogTitle>
                                                    <AlertDialogDescription>
                                                        Esta acción no se puede deshacer. Esto eliminará permanentemente la nómina pendiente con ID
                                                        <span className="font-mono p-1 bg-muted rounded-sm">{p.id}</span>.
                                                        Podrá cargarla de nuevo si es necesario.
                                                    </AlertDialogDescription>
                                                </AlertDialogHeader>
                                                <AlertDialogFooter>
                                                    <AlertDialogCancel>Cancelar</AlertDialogCancel>
                                                    <AlertDialogAction onClick={() => handleDeletePendingPayroll(p.id)}>
                                                        Sí, eliminar nómina
                                                    </AlertDialogAction>
                                                </AlertDialogFooter>
                                            </AlertDialogContent>
                                        </AlertDialog>
                                    </DropdownMenuContent>
                                    </DropdownMenu>
                                </TableCell>
                            </TableRow>
                        ))}
                         {pendingPayroll.length === 0 && <TableRow><TableCell colSpan={5} className="text-center text-muted-foreground py-4">No hay nóminas pendientes de pago.</TableCell></TableRow>}
                    </TableBody>
                </Table>
            </CardContent>
        </Card>
      </TabsContent>
      <TabsContent value="paid" className="mt-6">
        <Card>
            <CardHeader className="flex flex-row justify-between items-center">
                <div>
                  <CardTitle>Historial de Nóminas Pagadas</CardTitle>
                  <CardDescription>
                      Un registro de todas las nóminas que han sido procesadas y pagadas.
                  </CardDescription>
                </div>
                 <Button variant="outline" size="sm" onClick={() => shareOrExport(paidPayroll, 'Historial de Nóminas Pagadas', 'historial_nominas', 'pdf')}>
                    <Share2 className="mr-2 h-4 w-4"/> Compartir/Exportar
                </Button>
            </CardHeader>
            <CardContent>
                 <Table>
                    <TableHeader>
                        <TableRow>
                            <TableHead>ID Nómina</TableHead>
                            <TableHead>Período</TableHead>
                            <TableHead># Empleados</TableHead>
                            <TableHead className="text-right">Monto Total</TableHead>
                            <TableHead className="text-center">Reporte</TableHead>
                        </TableRow>
                    </TableHeader>
                    <TableBody>
                       {paidPayroll.map(p => (
                            <TableRow key={p.id}>
                                <TableCell><Badge variant="secondary">{p.id}</Badge></TableCell>
                                <TableCell>{p.period}</TableCell>
                                <TableCell>{p.payments.length}</TableCell>
                                <TableCell className="text-right">
                                    <div className="font-medium">{formatCurrency(p.totalAmountBs, 'VES')}</div>
                                    <div className="text-xs text-muted-foreground">{formatCurrency(p.totalAmountBs / bcvRate, 'USD')}</div>
                                </TableCell>
                                <TableCell className="text-center">
                                    <DropdownMenu>
                                    <DropdownMenuTrigger asChild>
                                        <Button size="icon" variant="ghost" className="h-8 w-8">
                                        <MoreVertical className="h-4 w-4" />
                                        </Button>
                                    </DropdownMenuTrigger>
                                    <DropdownMenuContent align="end">
                                        <DropdownMenuLabel>Exportar</DropdownMenuLabel>
                                        <DropdownMenuItem onClick={() => shareOrExport([p], `Reporte de Nómina ${p.id}`, `nomina_${p.id}`, 'pdf')}>
                                            <Share2 className="mr-2 h-4 w-4" /> Compartir PDF
                                        </DropdownMenuItem>
                                        <DropdownMenuItem onClick={() => shareOrExport([p], `Reporte de Nómina ${p.id}`, `nomina_${p.id}`, 'excel')}>
                                            <Share2 className="mr-2 h-4 w-4" /> Compartir Excel
                                        </DropdownMenuItem>
                                    </DropdownMenuContent>
                                    </DropdownMenu>
                                </TableCell>
                            </TableRow>
                        ))}
                        {paidPayroll.length === 0 && <TableRow><TableCell colSpan={5} className="text-center text-muted-foreground py-4">No hay nóminas pagadas en el historial.</TableCell></TableRow>}
                    </TableBody>
                </Table>
            </CardContent>
        </Card>
      </TabsContent>
       <TabsContent value="individual" className="mt-6">
        <IndividualHistory />
      </TabsContent>
    </Tabs>
  );
}
// =================================================================================
// FILE: src/app/dashboard/settings/page.tsx
'use client';
import { useState, useEffect } from 'react';
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
  CardFooter
} from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { useToast } from '@/hooks/use-toast';
import { Switch } from '@/components/ui/switch';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table';
import { KeyRound, Trash2, PlusCircle, Save } from 'lucide-react';
import { companyName as defaultCompanyName, companyRif as defaultCompanyRif, companyAddress as defaultCompanyAddress } from '@/lib/placeholder-data';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@/components/ui/dialog';

interface Admin {
    id: number;
    name: string;
    email: string;
    key: string;
}

const initialAdmins: Admin[] = [
    { id: 1, name: 'Admin Principal', email: 'admin@example.com', key: 'supersecretkey' },
    { id: 2, name: 'Contador', email: 'contador@example.com', key: 'contadorkey123' }
];

function AddAdminDialog({ onAdd }: { onAdd: (name: string, email: string) => void }) {
    const [isOpen, setIsOpen] = useState(false);
    const [name, setName] = useState('');
    const [email, setEmail] = useState('');

    const handleAdd = () => {
        onAdd(name, email);
        setName('');
        setEmail('');
        setIsOpen(false);
    }

    return (
        <Dialog open={isOpen} onOpenChange={setIsOpen}>
            <DialogTrigger asChild>
                <Button variant="outline" className="w-full">
                    <PlusCircle className="mr-2 h-4 w-4" /> Agregar Nuevo Administrador
                </Button>
            </DialogTrigger>
            <DialogContent>
                <DialogHeader>
                    <DialogTitle>Agregar Nuevo Administrador</DialogTitle>
                    <DialogDescription>Complete los datos para agregar un nuevo administrador.</DialogDescription>
                </DialogHeader>
                <div className="grid gap-4 py-4">
                    <div className="space-y-2">
                        <Label htmlFor="new-admin-name">Nombre</Label>
                        <Input id="new-admin-name" value={name} onChange={(e) => setName(e.target.value)} placeholder="Ej. Ana Vivas" />
                    </div>
                    <div className="space-y-2">
                        <Label htmlFor="new-admin-email">Email</Label>
                        <Input id="new-admin-email" type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="ej. ana.vivas@example.com" />
                    </div>
                </div>
                <DialogFooter>
                    <Button onClick={handleAdd}>Agregar Administrador</Button>
                </DialogFooter>
            </DialogContent>
        </Dialog>
    );
}

export default function SettingsPage() {
    const { toast } = useToast();

    // State for app personalization
    const [appName, setAppName] = useState('Nómina Global VE');
    const [appLogoUrl, setAppLogoUrl] = useState('');
    
    // State for company information
    const [companyName, setCompanyName] = useState(defaultCompanyName);
    const [companyRif, setCompanyRif] = useState(defaultCompanyRif);
    const [companyAddress, setCompanyAddress] = useState(defaultCompanyAddress);

    // State for security settings
    const [enableLoginReport, setEnableLoginReport] = useState(true);
    const [admins, setAdmins] = useState<Admin[]>(initialAdmins);

    // Load saved settings from localStorage on component mount
    useEffect(() => {
        const loadSettings = () => {
            const savedAppName = localStorage.getItem('appName');
            const savedAppLogoUrl = localStorage.getItem('appLogoUrl');
            const savedCompanyName = localStorage.getItem('companyName');
            const savedCompanyRif = localStorage.getItem('companyRif');
            const savedCompanyAddress = localStorage.getItem('companyAddress');
            const savedEnableLoginReport = localStorage.getItem('enableLoginReport');
            const savedAdmins = localStorage.getItem('admins');

            if (savedAppName) setAppName(savedAppName);
            if (savedAppLogoUrl) setAppLogoUrl(savedAppLogoUrl);
            if (savedCompanyName) setCompanyName(savedCompanyName);
            if (savedCompanyRif) setCompanyRif(savedCompanyRif);
            if (savedCompanyAddress) setCompanyAddress(savedCompanyAddress);
            if (savedEnableLoginReport) setEnableLoginReport(JSON.parse(savedEnableLoginReport));
            if (savedAdmins) setAdmins(JSON.parse(savedAdmins));
        }
        loadSettings();
    }, []);


    const handleSaveChanges = () => {
        localStorage.setItem('appName', appName);
        localStorage.setItem('appLogoUrl', appLogoUrl);
        localStorage.setItem('companyName', companyName);
        localStorage.setItem('companyRif', companyRif);
        localStorage.setItem('companyAddress', companyAddress);
        localStorage.setItem('enableLoginReport', JSON.stringify(enableLoginReport));
        localStorage.setItem('admins', JSON.stringify(admins));

        window.dispatchEvent(new Event('storage'));

        toast({
            title: "Ajustes Guardados",
            description: "La configuración de la aplicación ha sido actualizada.",
        });
    };
    
    const handleGenerateKey = (id: number) => {
        const newKey = `newkey_${Math.random().toString(36).substring(2, 10)}`;
        setAdmins(admins.map(admin => admin.id === id ? {...admin, key: newKey} : admin));
         toast({
            title: "Nueva Clave Generada",
            description: "Se ha generado una nueva clave de acceso para el administrador. Haga clic en 'Guardar Cambios' para aplicar.",
        });
    }

    const handleAddAdmin = (name: string, email: string) => {
        if (!name || !email) {
            toast({ variant: 'destructive', title: 'Error', description: 'Nombre y email son requeridos.' });
            return;
        }
        const newAdmin: Admin = {
            id: Date.now(),
            name,
            email,
            key: `key_${Math.random().toString(36).substring(2, 10)}`
        };
        setAdmins([...admins, newAdmin]);
        toast({ title: 'Administrador Agregado', description: "Haga clic en 'Guardar Cambios' para aplicar."});
    };

    const handleDeleteAdmin = (id: number) => {
        setAdmins(admins.filter(admin => admin.id !== id));
        toast({ title: 'Administrador Eliminado', description: "Haga clic en 'Guardar Cambios' para aplicar.", variant: 'destructive'});
    };

  return (
    <div className="space-y-6">
        <div className="flex items-center justify-between">
             <div>
                <h2 className="text-2xl font-bold tracking-tight">Ajustes Generales</h2>
                <p className="text-muted-foreground">
                    Personalice la aplicación y gestione la seguridad y los accesos.
                </p>
            </div>
            <Button onClick={handleSaveChanges}><Save className="mr-2 h-4 w-4"/> Guardar Todos los Cambios</Button>
        </div>

        <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
           <div className="space-y-6">
                {/* Company Information Card */}
                <Card>
                    <CardHeader>
                        <CardTitle>Información de la Empresa</CardTitle>
                        <CardDescription>Estos datos aparecerán en los recibos y reportes.</CardDescription>
                    </CardHeader>
                    <CardContent className="space-y-4">
                        <div className="space-y-2">
                            <Label htmlFor="company-name">Nombre de la Empresa</Label>
                            <Input id="company-name" value={companyName} onChange={(e) => setCompanyName(e.target.value)} />
                        </div>
                         <div className="space-y-2">
                            <Label htmlFor="company-rif">RIF</Label>
                            <Input id="company-rif" value={companyRif} onChange={(e) => setCompanyRif(e.target.value)} />
                        </div>
                        <div className="space-y-2">
                            <Label htmlFor="company-address">Dirección Fiscal</Label>
                            <Input id="company-address" value={companyAddress} onChange={(e) => setCompanyAddress(e.target.value)} />
                        </div>
                    </CardContent>
                </Card>

                {/* App Personalization Card */}
                <Card>
                    <CardHeader>
                        <CardTitle>Personalización de la App</CardTitle>
                        <CardDescription>Cambie el nombre y el logo de la aplicación.</CardDescription>
                    </CardHeader>
                    <CardContent className="space-y-4">
                        <div className="space-y-2">
                            <Label htmlFor="app-name">Nombre de la Aplicación</Label>
                            <Input id="app-name" value={appName} onChange={(e) => setAppName(e.target.value)} />
                        </div>
                        <div className="space-y-2">
                            <Label htmlFor="app-logo">URL del Logo</Label>
                            <Input id="app-logo" placeholder="https://example.com/logo.png" value={appLogoUrl} onChange={(e) => setAppLogoUrl(e.target.value)}/>
                            <p className="text-xs text-muted-foreground">Dejar en blanco para usar el logo por defecto.</p>
                        </div>
                    </CardContent>
                </Card>
           </div>

           <div className="space-y-6">
                {/* Access Management Card */}
                <Card>
                    <CardHeader>
                        <CardTitle>Gestión de Administradores</CardTitle>
                        <CardDescription>Añada, elimine y gestione claves de acceso.</CardDescription>
                    </CardHeader>
                    <CardContent>
                    <Table>
                            <TableHeader>
                                <TableRow>
                                    <TableHead>Nombre</TableHead>
                                    <TableHead>Clave de Acceso</TableHead>
                                    <TableHead className="text-right">Acciones</TableHead>
                                </TableRow>
                            </TableHeader>
                            <TableBody>
                                {admins.map(admin => (
                                    <TableRow key={admin.id}>
                                        <TableCell className="font-medium">{admin.name}<p className="text-xs text-muted-foreground">{admin.email}</p></TableCell>
                                        <TableCell className="font-mono text-sm">{admin.key}</TableCell>
                                        <TableCell className="text-right">
                                            <Button variant="ghost" size="icon" onClick={() => handleGenerateKey(admin.id)} title="Generar nueva clave"><KeyRound className="h-4 w-4" /></Button>
                                            <Button variant="ghost" size="icon" className="text-destructive" onClick={() => handleDeleteAdmin(admin.id)} title="Eliminar administrador"><Trash2 className="h-4 w-4" /></Button>
                                        </TableCell>
                                    </TableRow>
                                ))}
                            </TableBody>
                    </Table>
                    </CardContent>
                    <CardFooter>
                       <AddAdminDialog onAdd={handleAddAdmin} />
                    </CardFooter>
                </Card>

                {/* Security Settings Card */}
                <Card>
                    <CardHeader>
                        <CardTitle>Seguridad y Reportes</CardTitle>
                        <CardDescription>Ajustes relacionados a la seguridad del sistema.</CardDescription>
                    </CardHeader>
                    <CardContent>
                        <div className="flex items-center justify-between rounded-lg border p-4">
                            <div className="space-y-0.5">
                                <Label htmlFor="login-report" className="text-base">Habilitar Reporte de Accesos</Label>
                                <p className="text-sm text-muted-foreground">
                                    Registrar cada inicio de sesión de los administradores.
                                </p>
                            </div>
                            <Switch
                                id="login-report"
                                checked={enableLoginReport}
                                onCheckedChange={setEnableLoginReport}
                            />
                        </div>
                    </CardContent>
                </Card>
            </div>
        </div>
    </div>
  );
}
// =================================================================================
// FILE: src/components/compliance-checker.tsx
'use client';

import { useState, useRef, useEffect } from 'react';
import { complianceNotification } from '@/ai/flows/compliance-notification';
import { Button } from '@/components/ui/button';
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from '@/components/ui/card';
import { Textarea } from '@/components/ui/textarea';
import { Label } from '@/components/ui/label';
import { AlertCircle, Loader2, Terminal, FileUp, Camera, Text, VideoOff } from 'lucide-react';
import { Alert, AlertDescription, AlertTitle } from './ui/alert';
import { useToast } from '@/hooks/use-toast';
import { cn } from '@/lib/utils';
import Image from 'next/image';

const defaultLaws = `
- Ley Orgánica del Trabajo, los Trabajadores y las Trabajadoras (LOTTT)
- Ley Orgánica de Prevención, Condiciones y Medio Ambiente de Trabajo (LOPCYMAT)
- Decretos de aumento de salario mínimo e Ingreso Mínimo Integral.
- Ley del Seguro Social (IVSS)
- Ley del Régimen Prestacional de Vivienda y Hábitat (FAOV)
`.trim();

type InputMode = 'text' | 'file' | 'camera';

export function ComplianceChecker() {
  const [inputMode, setInputMode] = useState<InputMode>('text');
  const [employeeContract, setEmployeeContract] = useState('');
  const [photoDataUri, setPhotoDataUri] = useState<string | null>(null);
  const [relevantLaws, setRelevantLaws] = useState(defaultLaws);
  const [complianceIssues, setComplianceIssues] = useState<string[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [hasCameraPermission, setHasCameraPermission] = useState<boolean | null>(null);
  const { toast } = useToast();

  const fileInputRef = useRef<HTMLInputElement>(null);
  const videoRef = useRef<HTMLVideoElement>(null);
  const canvasRef = useRef<HTMLCanvasElement>(null);

  useEffect(() => {
    async function getCameraPermission() {
      if (inputMode !== 'camera' || hasCameraPermission) return;
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        if (videoRef.current) {
          videoRef.current.srcObject = stream;
        }
        setHasCameraPermission(true);
      } catch (error) {
        console.error('Error accessing camera:', error);
        setHasCameraPermission(false);
        toast({
          variant: 'destructive',
          title: 'Acceso a la cámara denegado',
          description:
            'Por favor, habilite los permisos de la cámara en su navegador para usar esta función.',
        });
      }
    }
    getCameraPermission();

    return () => {
      // Stop camera stream when component unmounts or mode changes
      if (videoRef.current && videoRef.current.srcObject) {
        const stream = videoRef.current.srcObject as MediaStream;
        stream.getTracks().forEach((track) => track.stop());
      }
    };
  }, [inputMode, hasCameraPermission, toast]);

  const handleFileChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    const file = event.target.files?.[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        setPhotoDataUri(e.target?.result as string);
        setEmployeeContract('');
      };
      reader.readAsDataURL(file);
    }
  };
  
  const takePicture = () => {
    if (videoRef.current && canvasRef.current) {
        const video = videoRef.current;
        const canvas = canvasRef.current;
        canvas.width = video.videoWidth;
        canvas.height = video.videoHeight;
        const context = canvas.getContext('2d');
        if (context) {
            context.drawImage(video, 0, 0, video.videoWidth, video.videoHeight);
            const dataUri = canvas.toDataURL('image/jpeg');
            setPhotoDataUri(dataUri);
            setEmployeeContract('');
        }
    }
  };


  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsLoading(true);
    setComplianceIssues([]);

    if (!employeeContract.trim() && !photoDataUri) {
      toast({
        variant: 'destructive',
        title: 'Error de validación',
        description: 'Por favor, proporcione un contrato para analizar.',
      });
      setIsLoading(false);
      return;
    }

    try {
      const result = await complianceNotification({
        employeeContract: employeeContract.trim() ? employeeContract : undefined,
        photoDataUri: photoDataUri ? photoDataUri : undefined,
        relevantLaws,
      });
      setComplianceIssues(result.complianceIssues);
    } catch (error) {
      console.error('Error checking compliance:', error);
      toast({
        variant: 'destructive',
        title: 'Error de Análisis',
        description:
          'No se pudo completar el análisis de cumplimiento. Por favor, intente de nuevo.',
      });
    } finally {
      setIsLoading(false);
    }
  };

  const renderInputArea = () => {
    switch (inputMode) {
      case 'text':
        return (
          <Textarea
            id="employee-contract"
            placeholder="Pegue aquí el texto completo del contrato del empleado..."
            className="h-48"
            value={employeeContract}
            onChange={(e) => {
              setEmployeeContract(e.target.value);
              setPhotoDataUri(null);
            }}
            disabled={isLoading}
          />
        );
      case 'file':
        return (
          <div className="flex flex-col items-center justify-center h-48 border-2 border-dashed rounded-lg">
            <input
              type="file"
              ref={fileInputRef}
              onChange={handleFileChange}
              className="hidden"
              accept="image/*"
              disabled={isLoading}
            />
            {photoDataUri ? (
              <div className="relative w-full h-full p-2">
                <Image src={photoDataUri} alt="Vista previa del contrato" layout="fill" objectFit="contain" />
              </div>
            ) : (
              <Button
                type="button"
                variant="outline"
                onClick={() => fileInputRef.current?.click()}
                disabled={isLoading}
              >
                <FileUp className="mr-2 h-4 w-4" />
                Seleccionar Archivo
              </Button>
            )}
          </div>
        );
      case 'camera':
        return (
          <div className="space-y-2">
             <div className="relative h-48 w-full bg-muted rounded-md overflow-hidden flex items-center justify-center">
                {photoDataUri ? (
                    <div className="relative w-full h-full">
                      <Image src={photoDataUri} alt="Foto del contrato" layout="fill" objectFit="contain" />
                    </div>
                ) : (
                    <>
                      <video ref={videoRef} className="w-full h-full object-cover" autoPlay muted playsInline />
                      {hasCameraPermission === false && (
                          <div className="absolute inset-0 flex flex-col items-center justify-center bg-black/50 text-white p-4">
                              <VideoOff className="h-8 w-8 mb-2" />
                              <p className="text-center">No se pudo acceder a la cámara. Revise los permisos de su navegador.</p>
                          </div>
                      )}
                    </>
                )}
             </div>
             <canvas ref={canvasRef} className="hidden" />
             <div className="flex justify-center gap-2">
                <Button type="button" onClick={takePicture} disabled={isLoading || !hasCameraPermission || !!photoDataUri}>
                    <Camera className="mr-2 h-4 w-4"/> Tomar Foto
                </Button>
                {photoDataUri && (
                    <Button type="button" variant="outline" onClick={() => setPhotoDataUri(null)} disabled={isLoading}>
                        Tomar de Nuevo
                    </Button>
                )}
             </div>
          </div>
        );
      default:
        return null;
    }
  };

  return (
    <div className="grid grid-cols-1 gap-8 lg:grid-cols-2">
      <form onSubmit={handleSubmit}>
        <Card>
          <CardHeader>
            <CardTitle>Verificador de Cumplimiento IA</CardTitle>
            <CardDescription>
              Analice contratos laborales contra la legislación venezolana para
              detectar posibles problemas de cumplimiento.
            </CardDescription>
          </CardHeader>
          <CardContent className="space-y-4">
             <div className="space-y-2">
                <Label>Fuente del Contrato</Label>
                <div className="flex gap-2">
                  <Button type="button" variant={inputMode === 'text' ? 'secondary' : 'outline'} onClick={() => setInputMode('text')} className="flex-1">
                    <Text className="mr-2 h-4 w-4"/> Pegar Texto
                  </Button>
                  <Button type="button" variant={inputMode === 'file' ? 'secondary' : 'outline'} onClick={() => setInputMode('file')} className="flex-1">
                    <FileUp className="mr-2 h-4 w-4"/> Subir Archivo
                  </Button>
                  <Button type="button" variant={inputMode === 'camera' ? 'secondary' : 'outline'} onClick={() => setInputMode('camera')} className="flex-1">
                    <Camera className="mr-2 h-4 w-4"/> Tomar Foto
                  </Button>
                </div>
            </div>
            
            <div className="space-y-2">
              <Label htmlFor="employee-contract">Contenido del Contrato</Label>
              {renderInputArea()}
            </div>

            <div className="space-y-2">
              <Label htmlFor="relevant-laws">Leyes Venezolanas Relevantes</Label>
              <Textarea
                id="relevant-laws"
                placeholder="Liste las leyes y decretos relevantes..."
                className="h-32"
                value={relevantLaws}
                onChange={(e) => setRelevantLaws(e.target.value)}
                disabled={isLoading}
              />
            </div>
          </CardContent>
          <CardFooter>
            <Button type="submit" disabled={isLoading} className="w-full">
              {isLoading ? (
                <>
                  <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                  Analizando...
                </>
              ) : (
                'Verificar Cumplimiento'
              )}
            </Button>
          </CardFooter>
        </Card>
      </form>

      <div className="space-y-4">
        <h2 className="text-2xl font-semibold tracking-tight">Resultados del Análisis</h2>
        <Card className="min-h-[400px]">
          <CardContent className="p-6">
            {isLoading ? (
              <div className="flex h-full min-h-[350px] flex-col items-center justify-center">
                <Loader2 className="h-12 w-12 animate-spin text-muted-foreground" />
                <p className="mt-4 text-muted-foreground">
                  La IA está analizando el documento...
                </p>
              </div>
            ) : complianceIssues.length > 0 ? (
              <Alert>
                <Terminal className="h-4 w-4" />
                <AlertTitle>Posibles Problemas de Cumplimiento Detectados</AlertTitle>
                <AlertDescription>
                  <ul className="mt-2 list-disc space-y-2 pl-5">
                    {complianceIssues.map((issue, index) => (
                      <li key={index}>{issue}</li>
                    ))}
                  </ul>
                </AlertDescription>
              </Alert>
            ) : (
              <div className="flex h-full min-h-[350px] flex-col items-center justify-center text-center">
                <AlertCircle className="h-12 w-12 text-muted-foreground" />
                <p className="mt-4 text-muted-foreground">
                  Los resultados del análisis de cumplimiento aparecerán aquí.
                  <br />
                  Envíe un contrato para comenzar.
                </p>
              </div>
            )}
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
// =================================================================================
// FILE: src/components/icons.tsx
import type { SVGProps } from 'react';

export function AppLogo(props: SVGProps<SVGSVGElement>) {
  return (
    <svg
      xmlns="http://www.w3.org/2000/svg"
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth="2"
      strokeLinecap="round"
      strokeLinejoin="round"
      {...props}
    >
      <path d="M12 12m-9 0a9 9 0 1 0 18 0a9 9 0 1 0 -18 0"></path>
      <path d="M12 12m-4 0a4 4 0 1 0 8 0a4 4 0 1 0 -8 0"></path>
      <path d="M12 2l0 2"></path>
      <path d="M12 20l0 2"></path>
      <path d="M20 12l2 0"></path>
      <path d="M2 12l2 0"></path>
    </svg>
  );
}
// =================================================================================
// FILE: src/components/receipt.tsx
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from '@/components/ui/card';
import { Separator } from '@/components/ui/separator';
import { Employee, Adjustment, Concept } from '@/lib/placeholder-data';
import { format, parseISO, lastDayOfMonth } from 'date-fns';
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger } from '@/components/ui/dropdown-menu';
import { Button } from '@/components/ui/button';
import { MoreVertical, Download } from 'lucide-react';

const formatCurrency = (amount: number, currency: 'USD' | 'VES') => {
  return new Intl.NumberFormat('es-VE', {
    style: 'currency',
    currency: currency,
    minimumFractionDigits: 2,
  }).format(amount);
};

export const ReportHeader = ({ companyName, companyRif, companyAddress }: { companyName: string, companyRif: string, companyAddress: string }) => (
    <div className="text-center mb-4">
      <h3 className="text-lg font-bold">{companyName}</h3>
      <p className="text-sm text-muted-foreground">RIF: {companyRif}</p>
      <p className="text-sm text-muted-foreground">{companyAddress}</p>
    </div>
);

export const ReportFooter = ({ employee, companyName, companyRif }: { employee: Employee | null, companyName: string, companyRif: string }) => (
    <>
      <Separator className="my-6" />
      <div className="grid grid-cols-2 gap-4 pt-8">
        <div className="flex flex-col items-center justify-center">
          <div className="w-4/5 border-t border-foreground"></div>
          <p className="mt-2 text-xs font-semibold">Firma del Trabajador</p>
          {employee && (
            <>
              <p className="text-xs text-muted-foreground">{employee.name}</p>
              <p className="text-xs text-muted-foreground">C.I. {employee.cedula}</p>
            </>
          )}
        </div>
        <div className="flex flex-col items-center justify-center">
          <div className="w-4/5 border-t border-foreground"></div>
          <p className="mt-2 text-xs font-semibold">Firma del Empleador</p>
          <p className="text-xs text-muted-foreground">{companyName}</p>
          <p className="text-xs text-muted-foreground">RIF: {companyRif}</p>
        </div>
      </div>
    </>
);

const getPeriodString = (year: string, month: string, fortnight: 'first' | 'second') => {
    if (!year || !month) return 'Período no definido';
    
    const monthNames = ["enero", "febrero", "marzo", "abril", "mayo", "junio", "julio", "agosto", "septiembre", "octubre", "noviembre", "diciembre"];
    const monthIndex = monthNames.findIndex(m => month.toLowerCase().startsWith(m.substring(0,3)));

    if (monthIndex === -1) return 'Mes no válido';
    
    const date = new Date(parseInt(year), monthIndex, 1);
    
    if (fortnight === 'first') {
        const startDay = '01';
        const endDay = '15';
        return `Período: ${startDay}/${format(date, 'MM/yyyy')} - ${endDay}/${format(date, 'MM/yyyy')}`;
    } else {
        const endOfMonth = lastDayOfMonth(date);
        const startDay = '16';
        const endDay = format(endOfMonth, 'dd');
        return `Período: ${startDay}/${format(date, 'MM/yyyy')} - ${endDay}/${format(date, 'MM/yyyy')}`;
    }
}

interface ReceiptProps {
    employee: Employee,
    adjustments: Adjustment[],
    companyName: string,
    companyRif: string,
    companyAddress: string,
    bcvRate: number,
    activePaymentConcepts: Concept[],
    activeDeductionConcepts: Concept[],
    payrollPeriod: { year: string, month: string, fortnight: 'first' | 'second' },
    onDownloadPdf?: () => void;
}

export function getReceiptData({ employee, adjustments, bcvRate, activePaymentConcepts, activeDeductionConcepts, payrollPeriod }: Omit<ReceiptProps, 'onDownloadPdf' | 'companyName' | 'companyRif' | 'companyAddress'>) {
  let totalAssignmentsBs = 0;
  let totalDeductionsBs = 0;

  const employeeAssignments = adjustments.filter(a => a.employeeCedula === employee.cedula && a.type === 'assignment');
  const employeeDeductions = adjustments.filter(a => a.employeeCedula === employee.cedula && a.type === 'deduction');

  const assignmentsList: { description: string, amountBs: number, note?: string }[] = [];
  
  const baseSalaryFortnightly = employee.baseSalaryBs / 2;
  assignmentsList.push({
      description: 'Salario Base Quincenal',
      amountBs: baseSalaryFortnightly
  });
  totalAssignmentsBs += baseSalaryFortnightly;

  activePaymentConcepts.forEach(concept => {
    if (!concept.active) return;
    if (concept.isSystemConcept && !concept.isEditable) return;
    
    let amountBs = 0;
    let note = '';
    
    const overrideAdjustment = employeeAssignments.find(a => a.conceptId === concept.id);

    if (overrideAdjustment) {
        amountBs = overrideAdjustment.amount;
        note = `Ajuste individual.`;
    } else {
        if (concept.amountType === 'USD') {
            amountBs = (concept.amountUsd || 0) * bcvRate;
            note = `${formatCurrency(concept.amountUsd || 0, 'USD')} a tasa ${formatCurrency(bcvRate, 'VES')}`
        } else {
            amountBs = concept.amountVes || 0;
        }

        if (concept.paymentFrequency === 'monthly') {
            amountBs = amountBs / 2;
            if (note) note += ' (pago quincenal)'; else note = 'Pago quincenal';
        }
    }

    if(amountBs > 0) {
      assignmentsList.push({
          description: concept.name,
          amountBs: amountBs,
          note
      });
      totalAssignmentsBs += amountBs;
    }
  });

  const manualAssignments = employeeAssignments.filter(a => a.conceptId.startsWith('manual-'));
  manualAssignments.forEach(adj => {
    assignmentsList.push({
        description: adj.description,
        amountBs: adj.amount
    });
    totalAssignmentsBs += adj.amount;
  });
  
  const deductionsList: { description: string, amountBs: number }[] = [];
  activeDeductionConcepts.forEach(concept => {
    if (!concept.active) return;
  });

  employeeDeductions.forEach(adj => {
    deductionsList.push({
        description: adj.description,
        amountBs: adj.amount,
    });
    totalDeductionsBs += adj.amount;
  });

  const netToPayBs = totalAssignmentsBs - totalDeductionsBs;
  const periodString = getPeriodString(payrollPeriod.year, payrollPeriod.month, payrollPeriod.fortnight);

  return {
    totalAssignmentsBs,
    totalDeductionsBs,
    assignmentsList,
    deductionsList,
    netToPayBs,
    periodString
  }
}

export function Receipt(props: ReceiptProps) {
  const { employee, companyName, companyRif, companyAddress, bcvRate, onDownloadPdf } = props;
  const { totalAssignmentsBs, totalDeductionsBs, assignmentsList, deductionsList, netToPayBs, periodString } = getReceiptData(props);

  return (
     <Card className="overflow-hidden">
          <CardHeader className="bg-muted/50 p-6 space-y-4">
            <div className="flex items-start justify-between">
                <div className='flex-1'>
                    <ReportHeader companyName={companyName} companyRif={companyRif} companyAddress={companyAddress} />
                </div>
                {onDownloadPdf && (
                    <DropdownMenu>
                        <DropdownMenuTrigger asChild>
                            <Button variant="ghost" size="icon" className="h-8 w-8 ml-auto">
                                <MoreVertical className="h-4 w-4" />
                            </Button>
                        </DropdownMenuTrigger>
                        <DropdownMenuContent align="end">
                            <DropdownMenuItem onClick={onDownloadPdf}>
                                <Download className="mr-2 h-4 w-4" />
                                Descargar PDF
                            </DropdownMenuItem>
                        </DropdownMenuContent>
                    </DropdownMenu>
                )}
            </div>
            <div className="flex items-start justify-between">
              <div>
                <CardTitle>Recibo de Pago Quincenal</CardTitle>
                <CardDescription>
                  {periodString} | Tasa BCV: {formatCurrency(bcvRate, 'VES')}
                </CardDescription>
              </div>
              <div className="text-right">
                  <p className="font-semibold">{employee.name}</p>
                  <p className="text-sm text-muted-foreground">
                    C.I. {employee.cedula}
                  </p>
              </div>
            </div>
             <Separator />
            <div className="grid grid-cols-2 gap-x-4 text-sm">
                <div><span className="font-semibold">Cargo:</span> {employee.position}</div>
                <div><span className="font-semibold">Fecha de Ingreso:</span> {format(parseISO(employee.hireDate), 'dd/MM/yyyy')}</div>
            </div>
          </CardHeader>
          <CardContent className="p-6 text-sm">
            <div className="grid grid-cols-3 gap-4">
              <div className="space-y-2 col-span-2">
                <h3 className="font-semibold text-primary">Asignaciones</h3>
                 {assignmentsList.map((item, i) => (
                    <div className="flex justify-between" key={`assign-list-${i}`}>
                        <div>
                            <span>{item.description}</span>
                            {item.note && <p className="text-xs text-muted-foreground">{item.note}</p>}
                        </div>
                        <div className="text-right">
                            <p>{formatCurrency(item.amountBs, 'VES')}</p>
                            <p className="text-xs text-muted-foreground">{formatCurrency(item.amountBs / bcvRate, 'USD')}</p>
                        </div>
                    </div>
                 ))}
              </div>
              <div className="space-y-2">
                <h3 className="font-semibold text-destructive">Deducciones</h3>
                {deductionsList.map((adj, i) => (
                    <div className="flex justify-between" key={`deduct-${i}`}>
                        <span>{adj.description}</span>
                        <div className="text-right">
                            <p>{formatCurrency(-adj.amountBs, 'VES')}</p>
                            <p className="text-xs text-muted-foreground">{formatCurrency(-adj.amountBs / bcvRate, 'USD')}</p>
                        </div>
                    </div>
                ))}
                {deductionsList.length === 0 && (
                    <p className="text-xs text-muted-foreground">No hay deducciones para este período.</p>
                )}
              </div>
            </div>
            <Separator className="my-4" />
            <div className="grid grid-cols-3 gap-4 font-semibold">
              <div className="col-span-2">
                <div className="flex justify-between">
                  <span>Total Asignaciones</span>
                  <div className="text-right">
                    <p>{formatCurrency(totalAssignmentsBs, 'VES')}</p>
                    <p className="text-xs font-normal text-muted-foreground">{formatCurrency(totalAssignmentsBs / bcvRate, 'USD')}</p>
                  </div>
                </div>
              </div>
              <div>
                <div className="flex justify-between">
                  <span>Total Deducciones</span>
                   <div className="text-right">
                    <p>{formatCurrency(-totalDeductionsBs, 'VES')}</p>
                    <p className="text-xs font-normal text-muted-foreground">{formatCurrency(-totalDeductionsBs / bcvRate, 'USD')}</p>
                  </div>
                </div>
              </div>
            </div>
            <ReportFooter employee={employee} companyName={companyName} companyRif={companyRif} />
          </CardContent>
          <CardFooter className="bg-muted/50 p-6">
            <div className="flex w-full items-center justify-between">
              <h3 className="text-lg font-bold">NETO A PAGAR</h3>
               <div className="text-right">
                <p className="text-xl font-bold text-primary">{formatCurrency(netToPayBs, 'VES')}</p>
                <p className="text-base font-semibold text-muted-foreground">{formatCurrency(netToPayBs / bcvRate, 'USD')}</p>
              </div>
            </div>
          </CardFooter>
        </Card>
  )
}
// =================================================================================
// FILE: src/components/ui/toaster.tsx
"use client"

import { useToast } from "@/hooks/use-toast"
import {
  Toast,
  ToastClose,
  ToastDescription,
  ToastProvider,
  ToastTitle,
  ToastViewport,
} from "@/components/ui/toast"

export function Toaster() {
  const { toasts } = useToast()

  return (
    <ToastProvider>
      {toasts.map(function ({ id, title, description, action, ...props }) {
        return (
          <Toast key={id} {...props}>
            <div className="grid gap-1">
              {title && <ToastTitle>{title}</ToastTitle>}
              {description && (
                <ToastDescription>{description}</ToastDescription>
              )}
            </div>
            {action}
            <ToastClose />
          </Toast>
        )
      })}
      <ToastViewport />
    </ToastProvider>
  )
}

'use server';

/**
 * @fileOverview This file contains the Genkit flow for AI-powered compliance notifications.
 *
 * - complianceNotification - A function that triggers the compliance check and returns notifications.
 * - ComplianceNotificationInput - The input type for the complianceNotification function.
 * - ComplianceNotificationOutput - The return type for the complianceNotification function.
 */

import {ai} from '@/ai/genkit';
import {z} from 'genkit';

const ComplianceNotificationInputSchema = z.object({
  employeeContract: z
    .string()
    .optional()
    .describe('The full text of the employee contract.'),
  photoDataUri: z
    .string()
    .optional()
    .describe(
      "A photo of a plant, as a data URI that must include a MIME type and use Base64 encoding. Expected format: 'data:<mimetype>;base64,<encoded_data>'."
    ),
  relevantLaws: z
    .string()
    .describe(
      'A list of relevant Venezuelan laws, including LOPCYMATT and any applicable executive decrees.'
    ),
});
export type ComplianceNotificationInput = z.infer<
  typeof ComplianceNotificationInputSchema
>;

const ComplianceNotificationOutputSchema = z.object({
  complianceIssues: z
    .array(z.string())
    .describe(
      'A list of potential compliance issues identified in the employee contract, based on the provided laws.'
    ),
});
export type ComplianceNotificationOutput = z.infer<
  typeof ComplianceNotificationOutputSchema
>;

export async function complianceNotification(
  input: ComplianceNotificationInput
): Promise<ComplianceNotificationOutput> {
  return complianceNotificationFlow(input);
}

const complianceNotificationPrompt = ai.definePrompt({
  name: 'complianceNotificationPrompt',
  input: {schema: ComplianceNotificationInputSchema},
  output: {schema: ComplianceNotificationOutputSchema},
  prompt: `You are an AI-powered legal assistant specializing in Venezuelan labor law.

  Your task is to review an employee contract and identify any potential compliance issues based on the provided relevant laws, including LOPCYMATT and executive decrees.
  
  The contract to be analyzed is provided below, either as text or as an image. Extract the relevant text.

  {{#if employeeContract}}
  Employee Contract Text:
  {{employeeContract}}
  {{/if}}

  {{#if photoDataUri}}
  Employee Contract Image:
  {{media url=photoDataUri}}
  {{/if}}

  Relevant Laws:
  {{relevantLaws}}

  Identify any clauses in the contract that may violate or contradict these laws. Provide a detailed list of compliance issues.

  Output should only be the list of issues.
  `,
});

const complianceNotificationFlow = ai.defineFlow(
  {
    name: 'complianceNotificationFlow',
    inputSchema: ComplianceNotificationInputSchema,
    outputSchema: ComplianceNotificationOutputSchema,
  },
  async input => {
    if (!input.employeeContract && !input.photoDataUri) {
      throw new Error('Either employeeContract or photoDataUri must be provided.');
    }
    const {output} = await complianceNotificationPrompt(input);
    return output!;
  }
);
