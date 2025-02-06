<script setup>
import { ref, watch } from "vue";
import jsPDF from "jspdf";

// URL del logo
const logoUrl = "https://app.iosep.gob.ar/Iosep/Content/Imagenes/logo__01Ar.png";

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

const getBase64Image = async (url) => {
  try {
    const response = await fetch(url);
    const blob = await response.blob();
    return new Promise((resolve) => {
      const reader = new FileReader();
      reader.onloadend = () => resolve(reader.result);
      reader.readAsDataURL(blob);
    });
  } catch (error) {
    console.error("Error al convertir la imagen a Base64:", error);
    return null;
  }
};

const generatePDF = async () => {
  const doc = new jsPDF();
  const logoBase64 = await getBase64Image(logoUrl);

  if (logoBase64) {
    doc.addImage(logoBase64, "PNG", 10, 10, 50, 20);
  } else {
    console.warn("No se pudo cargar el logo, se generará el PDF sin él.");
  }

  doc.setFontSize(16);
  doc.text("Declaración Jurada del Afiliado", 70, 20);

  let y = 40;
  doc.setFontSize(12);
  Object.keys(formData.value).forEach((key) => {
    doc.text(`${key.replace(/_/g, " ")}: ${formData.value[key] || "-"}`, 10, y);
    y += 10;
  });

  doc.save("declaracion_jurada.pdf");
};
</script>

<template>
  <div class="max-w-2xl mx-auto p-6 shadow-lg rounded-xl bg-white w-full min-h-[600px] flex flex-col justify-between border">
    <div class="flex justify-center mb-4">
      <img :src="logoUrl" alt="Logo" class="h-16">
    </div>
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
