<script setup>
import { ref, watch } from "vue";
import jsPDF from "jspdf";

const steps = ref([
  { id: 1, title: "1", fields: [
      { label: "Apellido y Nombre", type: "text", name: "apellido_nombre" },
      { label: "Apellido Paterno", type: "text", name: "apellido_paterno" },
      { label: "Apellido Materno", type: "text", name: "apellido_materno" },
      { label: "N° Afiliado", type: "text", name: "numero_afiliado" },
      { label: "Nacimiento", type: "date", name: "fecha_nacimiento" },
      { label: "Nacionalidad", type: "text", name: "nacionalidad" },
      { label: "Estado Civil", type: "select", name: "estado_civil", options: ["Soltero", "Casado", "Divorciado", "Viudo"] },
      { label: "Lugar Nac.", type: "text", name: "lugar_nacimiento" },
      { label: "Provincia", type: "text", name: "provincia" },
      { label: "DNI", type: "text", name: "documento_identidad" },
      { label: "CUIL", type: "text", name: "cuil" },
      { label: "Teléfono", type: "text", name: "telefono" },
  ]},
  { id: 2, title: "Resumen" },
  { id: 3, title: "Finalizado" }
]);

const currentStep = ref(0);
const formData = ref(JSON.parse(localStorage.getItem("formData")) || {});

const handleNext = () => {
  if (currentStep.value < steps.value.length - 1) currentStep.value++;
};
const handlePrev = () => {
  if (currentStep.value > 0) currentStep.value--;
};

watch(formData, (newData) => {
  localStorage.setItem("formData", JSON.stringify(newData));
}, { deep: true });

const generatePDF = () => {
  const doc = new jsPDF();

  // Estilo del título
  doc.setFont("helvetica", "bold");
  doc.setFontSize(24);
  doc.setTextColor(30, 144, 255); // Azul degradado
  doc.text("Declaración Jurada del Afiliado", 15, 20);

  // Línea separadora
  doc.setDrawColor(100, 100, 100);
  doc.line(15, 25, 195, 25);

  // Espaciado para los datos
  let y = 35;
  doc.setFont("helvetica", "normal");
  doc.setFontSize(12);
  doc.setTextColor(0, 0, 0); // Texto negro

  // Agrupar datos en dos columnas
  const keys = Object.keys(formData.value);
  const mitad = Math.ceil(keys.length / 2);

  for (let i = 0; i < mitad; i++) {
    const keyLeft = keys[i].replace(/_/g, " ");
    const valueLeft = formData.value[keys[i]] || "-";
    const keyRight = keys[i + mitad] ? keys[i + mitad].replace(/_/g, " ") : "";
    const valueRight = keys[i + mitad] ? formData.value[keys[i + mitad]] || "-" : "";

    doc.text(`${keyLeft}:`, 15, y);
    doc.text(`${valueLeft}`, 65, y);

    if (keyRight) {
      doc.text(`${keyRight}:`, 110, y);
      doc.text(`${valueRight}`, 160, y);
    }

    y += 10;
  }

  // Línea final separadora
  doc.setDrawColor(100, 100, 100);
  doc.line(15, y, 195, y);

  doc.save("declaracion_jurada.pdf");
};
</script>

<template>
  <div class="max-w-2xl mx-auto p-6 shadow-lg rounded-xl bg-white w-full min-h-[600px] flex flex-col justify-between border">
    <h1 class="text-2xl font-bold text-center bg-gradient-to-r from-blue-500 to-green-500 text-transparent bg-clip-text mb-4">
      Declaración Jurada del Afiliado
    </h1>
    <transition name="fade" mode="out-in">
      <div :key="currentStep" class="flex-grow">
        <div v-if="currentStep === 0">
          <div class="grid grid-cols-1 gap-3 sm:grid-cols-2 md:grid-cols-3">
            <div v-for="field in steps[currentStep].fields" :key="field.name" class="w-full">
              <label class="block text-gray-700 text-sm font-semibold">{{ field.label }}</label>
              <input v-if="field.type !== 'select'" :type="field.type" v-model="formData[field.name]" class="w-full p-2 border border-gray-300 rounded">
              <select v-else v-model="formData[field.name]" class="w-full p-2 border border-gray-300 rounded">
                <option v-for="option in field.options" :key="option" :value="option">{{ option }}</option>
              </select>
            </div>
          </div>
        </div>
        <div v-else-if="currentStep === 1">
          <h2 class="text-xl font-bold mb-2">Resumen</h2>
          <div class="bg-gray-100 p-4 rounded-lg shadow-sm">
            <div v-for="(value, key) in formData" :key="key" class="mb-2 flex justify-between border-b pb-2">
              <span class="font-semibold text-gray-700">{{ key.replace(/_/g, " ") }}:</span>
              <span class="text-gray-600">{{ value || "-" }}</span>
            </div>
          </div>
        </div>
        <div v-else>
          <button @click="generatePDF" class="bg-green-500 text-white px-4 py-2 rounded-lg text-sm mx-auto block">
            Descargar PDF
          </button>
        </div>
      </div>
    </transition>
    <div class="flex justify-between mt-6 bg-white py-2">
      <button v-if="currentStep > 0" @click="handlePrev" class="bg-gray-300 text-gray-700 px-3 py-2 rounded-lg text-sm">
        Atrás
      </button>
      <button v-if="currentStep < steps.length - 1" @click="handleNext" class="bg-blue-500 text-white px-4 py-2 rounded-lg text-sm">
        Siguiente
      </button>
    </div>
  </div>
</template>

<style>
body {
  margin: 0;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background-color: #f5f5f5;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
  transform: translateY(10px);
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
  border: 1px solid #ccc;
  border-radius: 0.375rem;
  width: 100%;
}

button {
  font-size: 0.875rem;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 0.375rem;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

button.bg-green-500 {
  background-color: #28a745;
  color: #fff;
}

button.bg-green-500:hover {
  background-color: #1c7c31;
}
</style>
