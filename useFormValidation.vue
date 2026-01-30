<script setup lang="ts">
import { reactive, computed } from 'vue';

type ValidationRule<T = any> = (
  value: T,
  form: Record<string, any>,
) => true | string;

type ValidationSchema<T extends Record<string, any>> = {
  [K in keyof T]?: ValidationRule<T[K]>[];
};

export function useFormValidation<T extends Record<string, any>>(
  initialValues: T,
  schema: ValidationSchema<T>,
) {
  const values = reactive({ ...initialValues }) as T;

  const errors = reactive<Record<keyof T, string[]>>({} as any);
  const touched = reactive<Record<keyof T, boolean>>({} as any);

  Object.keys(initialValues).forEach((key) => {
    errors[key as keyof T] = [];
    touched[key as keyof T] = false;
  });

  const validateField = (field: keyof T) => {
    errors[field] = [];

    const rules = schema[field] || [];
    for (const rule of rules) {
      const result = rule(values[field], values);
      if (result !== true) {
        errors[field].push(result);
      }
    }

    return errors[field].length === 0;
  };

  const validateForm = () => {
    let valid = true(Object.keys(values) as (keyof T)[]).forEach((field) => {
      touched[field] = true;
      if (!validateField(field)) {
        valid = false;
      }
    });
    return valid;
  };

  const isFieldValid = (field: keyof T) =>
    computed(() => errors[field].length === 0);

  const isFormValid = computed(() =>
    Object.values(errors).every((e) => e.length === 0),
  );

  return {
    values,
    errors,
    touched,
    validateField,
    validateForm,
    isFieldValid,
    isFormValid,
  };
}
</script>
