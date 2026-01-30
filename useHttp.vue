<script setup lang="ts">
import { ref } from 'vue';

type HttpMethod = 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE';

interface UseHttpOptions<TBody = any> {
  url: string;
  method?: HttpMethod;
  headers?: HeadersInit;
  body?: TBody;
  immediate?: boolean;
}

export function useHttp<TResponse = any, TBody = any>(
  options: UseHttpOptions<TBody>,
) {
  const data = ref<TResponse | null>(null);
  const error = ref<string | null>(null);
  const status = ref<number | null>(null);

  const loading = ref(false);
  const success = ref(false);

  const execute = async (override?: Partial<UseHttpOptions<TBody>>) => {
    loading.value = true;
    success.value = false;
    error.value = null;

    try {
      const response = await fetch(override?.url || options.url, {
        method: override?.method || options.method || 'GET',
        headers: {
          'Content-Type': 'application/json',
          ...options.headers,
          ...override?.headers,
        },
        body: options.body
          ? JSON.stringify(override?.body ?? options.body)
          : undefined,
      });

      status.value = response.status;

      if (!response.ok) {
        throw new Error(`HTTP error: ${response.status}`);
      }

      data.value = await response.json();
      success.value = true;
      return data.value;
    } catch (e: any) {
      error.value = e.message || 'Unknown error';
      throw e;
    } finally {
      loading.value = false;
    }
  };

  if (options.immediate) {
    execute();
  }

  return {
    data,
    error,
    status,
    loading,
    success,
    execute,
  };
}
</script>
