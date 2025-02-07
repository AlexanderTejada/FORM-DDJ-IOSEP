<template>
  <div class="max-w-7xl mx-auto p-6 shadow-lg rounded-xl bg-white w-full min-h-[600px] flex flex-col justify-between border subtle-gradient">   
    <!-- Logo -->
    <div class="flex justify-center mb-4">
      <img src="https://app.iosep.gob.ar/Iosep/Content/Imagenes/logo__01Ar.png" alt="Logo" class="h-16">
    </div>

    <h1 class="text-2xl font-extrabold text-center bg-gradient-to-r from-blue-900 to-blue-300 text-transparent bg-clip-text mb-5">
      DECLARACIÓN JURADA DE AFILIADO
    </h1>

    <transition name="fade" mode="out-in">
      <div v-if="currentStep === 0">
        <div class="space-y-6">
          <!-- Iteramos por las secciones -->
          <div v-for="section in groupedFields" :key="section.title">
            <h2 class="text-lg font-bold border-b-2 pb-2 border-gray-400 mb-3">{{ section.title }}</h2>
            <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-4">
              <div v-for="field in section.fields" :key="field.name" class="w-full">
                <label class="block text-gray-700 text-sm font-semibold">{{ field.label }}</label>
                <input 
                  v-if="field.type !== 'select'" 
                  :type="field.type" 
                  v-model="formData[field.name]" 
                  :maxlength="field.maxLength"
                  class="w-full p-2 border border-gray-300 rounded"
                />
                <select 
                  v-else 
                  v-model="formData[field.name]" 
                  class="w-full p-2 border border-gray-300 rounded"
                >
                  <option v-for="option in field.options" :key="option" :value="option">{{ option }}</option>
                </select>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div v-else-if="currentStep === 1">
        <h2 class="text-xl font-bold mb-2">Resumen</h2>
        <p class="text-gray-500">Por favor verifica que los datos que ingresaste sean correctos.</p>
        <div class="bg-gray-50 p-4 rounded-lg shadow-sm overflow-auto max-h-[400px]">
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
            <div class="col-span-2 border border-gray-300 rounded-lg p-4" v-for="section in groupedFields" :key="section.title">
              <h3 class="font-bold text-lg mb-2 border-b pb-2">{{ section.title }}</h3>
              <div v-for="field in section.fields" :key="field.name" class="mb-2 flex justify-between">
                <span class="font-semibold text-gray-700">{{ field.label }}:</span>
                <span class="text-gray-600">{{ formData[field.name] || "-" }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div v-else>
        <button @click="generatePDF" class="bg-blue-200 text-blue-400 px-6 py-4 rounded-lg text-lg mx-auto block flex flex-col items-center justify-center shadow-md hover:bg-blue-300 hover:shadow-lg transition-all duration-300">
          <img src="/dow_pdf.png" alt="Descargar PDF" class="h-18 w-10 mb-2"> 
          <span class="tracking-wide">Descargar</span>
        </button>
      </div>
    </transition>

    <div class="flex justify-between mt-6 bg-white py-2">
      <button v-if="currentStep > 0" @click="handlePrev" class="bg-gray-300 text-gray-700 px-3 py-2 rounded-lg text-sm">Atrás</button>
      <button v-if="currentStep < 2" @click="handleNext" 
        class="bg-gradient-to-r from-blue-500 to-blue-700 hover:from-blue-600 hover:to-blue-800 text-white font-medium py-2 px-4 rounded-lg transition duration-300 ease-in-out shadow-sm hover:shadow-md">
        Siguiente
      </button>
    </div>
  </div>
</template>
<script setup>
import { ref, watch } from "vue";
import jsPDF from "jspdf";

const currentStep = ref(0);
const formData = ref(JSON.parse(localStorage.getItem("formData")) || {});

const groupedFields = ref([
  {
    title: "Información Personal",
    fields: [
      { label: "Apellido y Nombre", type: "text", name: "nombre_completo", maxLength: 50 },
      { label: "Apellido Paterno", type: "text", name: "apellido_paterno", maxLength: 30 },
      { label: "Apellido Materno", type: "text", name: "apellido_materno", maxLength: 30 },
      { label: "Fecha de Nacimiento", type: "date", name: "fecha_nacimiento" },
      { label: "Nacionalidad", type: "text", name: "nacionalidad", maxLength: 30 },
      { label: "Estado Civil", type: "select", name: "estado_civil", options: ["Soltero", "Casado", "Divorciado", "Viudo"] },
    ]
  },
  {
    title: "Lugar de Nacimiento",
    fields: [
      { label: "Localidad de Nacimiento", type: "text", name: "localidad_nacimiento", maxLength: 50 },
      { label: "Provincia de Nacimiento", type: "text", name: "provincia_nacimiento", maxLength: 50 },
    ]
  },
  {
    title: "Información Laboral",
    fields: [
      { label: "Repartición", type: "text", name: "reparticion", maxLength: 100 },
      { label: "Ministerio", type: "text", name: "ministerio", maxLength: 100 },
      { label: "Fecha de Ingreso a la Administración", type: "date", name: "fecha_ingreso_administracion" },
    ]
  },
  {
    title: "Lugar de Pago",
    fields: [
      { label: "Lugar de Pago", type: "text", name: "lugar_pago", maxLength: 100 },
      { label: "Código del Lugar de Pago", type: "text", name: "codigo_lugar_pago", maxLength: 10 },
    ]
  },
  {
    title: "Domicilio Actual",
    fields: [
      { label: "Calle", type: "text", name: "domicilio_calle", maxLength: 100 },
      { label: "Número", type: "text", name: "domicilio_numero", maxLength: 10 },
      { label: "Departamento", type: "text", name: "domicilio_departamento", maxLength: 10 },
      { label: "Localidad", type: "text", name: "domicilio_localidad", maxLength: 50 },
      { label: "Provincia", type: "text", name: "domicilio_provincia", maxLength: 50 },
    ]
  },
  {
    title: "Datos de Contacto",
    fields: [
      { label: "Teléfono", type: "text", name: "telefono", maxLength: 15 },
      { label: "CUIL", type: "text", name: "cuil", maxLength: 10 },
      { label: "DNI", type: "text", name: "dni", maxLength: 8 },
      { label: "N° Afiliado", type: "text", name: "numero_afiliado", maxLength: 20 },
    ]
  }
]);

// Función para generar PDF
const generatePDF = async () => {
  const doc = new jsPDF();

  // Agregar Logo
  const img = new Image();
  img.src = "src/components/image.png"; // Ruta de la imagen
  doc.addImage(img, "PNG", 10, 10, 50, 20);

  // Agregar Título con Cinta de Color
  doc.setFillColor(33, 150, 243); // Azul tipo seguro médico
  doc.rect(10, 35, 190, 10, "F");
  doc.setTextColor(255, 255, 255);
  doc.setFontSize(16);
  doc.setFont("bold");
  doc.text("DECLARACIÓN JURADA DE AFILIADO", 60, 42);

  // Estilo de los datos
  doc.setTextColor(0, 0, 0);
  doc.setFontSize(12);
  doc.setFont("normal");

  let y = 55; // Posición vertical inicial
  groupedFields.value.forEach((section) => {
    if (y > 270) {
      doc.addPage();
      y = 10;
    }

    // Títulos de las secciones
    doc.setFontSize(13);
    doc.setFont("bold");
    doc.setTextColor(50, 50, 50);
    doc.text(section.title, 15, y);
    doc.setDrawColor(0, 0, 0);
    doc.line(15, y + 2, 195, y + 2); // Línea separadora
    y += 7;

    // Datos organizados en filas con colores
    doc.setFontSize(12);
    doc.setFont("normal");
    doc.setTextColor(80, 80, 80);
    
    section.fields.forEach((field, index) => {
      if (y > 270) {
        doc.addPage();
        y = 10;
      }
      
      // Alternar color de fondo para cada fila
      if (index % 2 === 0) {
        doc.setFillColor(240, 240, 240); // Gris claro
        doc.rect(15, y - 4, 180, 7, "F");
      }

      // Etiqueta
      doc.setFont("bold");
      doc.text(`${field.label}:`, 20, y);
      
      // Respuesta (sin línea debajo)
      doc.setFont("normal");
      doc.text(`${formData.value[field.name] || "-"}`, 90, y);
      
      y += 8;
    });

    y += 5;
  });

  // Firmas al final
  y = y < 240 ? 240 : y + 20;
  doc.setFontSize(12);
  doc.setFont("bold");
  doc.line(20, y + 5, 80, y + 5); // Línea de firma

  doc.text("Firma", 20, y);

  // Descargar PDF
  doc.save("declaracion_jurada.pdf");
};

const handleNext = () => currentStep.value++;
const handlePrev = () => currentStep.value--;

watch(formData, (newData) => {
  localStorage.setItem("formData", JSON.stringify(newData));
}, { deep: true });

</script>



<style>
body {
  margin: 0;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background: linear-gradient(to bottom, #546e7a, #78909c);

}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
  transform: translateY(10px);
}
.subtle-gradient {
  background: linear-gradient(to bottom, #eef6ff, #ffffff, #ffffff);
     
}

h1 {
  font-size: 1.5rem;
  color: #333;
}

label {
  font-size: 0.875rem;
  color: #555;
}

input, select {
  font-size: 0.875rem;
  padding: 0.5rem;
  border: 1px solid #3e086b;
  border-radius: 0.375rem;
  width: 100%;
}
button.bg-green-500 {
  background-color: #007bff; /* Azul */
  color: #fff;
  text-transform: uppercase;
  font-size: 1rem;
  font-family: sans-serif;
  border-radius: 8px;
  padding: 12px 24px;
  border: none;
  cursor: pointer;
  display: flex; /* Para alinear icono y texto */
  align-items: center;
  justify-content: center;
  transition: background-color 0.3s ease;
}

button.bg-green-500 i {
  margin-right: 8px; /* Espacio entre icono y texto */
}

button.bg-green-500:hover {
  background-color: #0056b3; /* Azul más oscuro */
}
</style>
