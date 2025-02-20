# TypeScript مع الواجهة التركيبية {#typescript-with-composition-api}

> لقراءة هذه الصفحة يجب عليك أولا الاطلاع على  [استخدام Vue مع TypeScript](./overview).

## إضافة الأنواع لخاصيات المكون {#typing-component-props}

### استخدام `<script setup>` {#using-script-setup}

عند استخدام `<script setup>`، تدعم التعليمة العامة `()defineProps` تحديد أنواع الخاصيات على أساس الوسيط الذي مُرر:

```vue
<script setup lang="ts">
const props = defineProps({
  foo: { type: String, required: true },
  bar: Number
})

props.foo // string
props.bar // number | undefined
</script>
```

هذا ما يُسمى بـ "التصريح أثناء زمن التشغيل"، لأن الوسيط المُمرر إلى `()defineProps` سيُستخدم كخيار `props` في زمن التشغيل.

ومع ذلك، فإنه عادة ما يكون أسهل تعريف الخاصيات باستخدام أنواع خالصة عبر وسيط معمم:

```vue
<script setup lang="ts">
const props = defineProps<{
  foo: string
  bar?: number
}>()
</script>
```

هذا ما يُسمى بـ "التصريح على أساس الأنواع"، وسيحاول المصرف أن يفعل ما يمكنه من أجل تحديد خيارات زمن التشغيل المُقابلة للوسيط المعمم. في هذه الحالة، سيصرف المثال الثاني والأول بالضبط إلى خيارات زمن التشغيل المُماثلة.

يمكنك استخدام أو التصريح على أساس الأنواع أو التصريح أثناء زمن التشغيل، لكن لا يمكنك استخدامهما معا في نفس الوقت.

يمكننا أيضا نقل أنواع الخاصيات إلى نوع واجهة منفصل:

```vue
<script setup lang="ts">
interface Props {
  foo: string
  bar?: number
}

const props = defineProps<Props>()
</script>
```

This also works if `Props` is imported from an external source. This feature requires TypeScript to be a peer dependency of Vue.

```vue
<script setup lang="ts">
import type { Props } from './foo'

const props = defineProps<Props>()
</script>
```

#### محدوديات الصيغة {#syntax-limitations}

في النسخة 3.2 وأقل، كان الوسيط النوعي لـ `()defineProps` محدودًا بالنوع الحرفي أو مرجع إلى واجهة محلية.

حُل هذا القيد في الإصدار 3.3. تدعم أحدث إصدارات Vue الإشارة إلى الأنواع المستوردة ومجموعة محدودة من الأنواع المعقدة في موضع وسيط النوع. ومع ذلك، لأن تحويل النوع إلى زمن التشغيل لا يزال يعتمد على AST، فإن بعض الأنواع المعقدة التي تتطلب تحليل النوع الفعلي ، على سبيل المثال الأنواع الشرطية، لا تُدعم. يمكنك استخدام الأنواع الشرطية لنوع خاصية واحدة ، ولكن ليس كائن الخاصيات بأكمله.

### القيم الافتراضية للخاصيات {#props-default-values}

عند استخدام التصريح على أساس الأنواع، فإننا نفقد القدرة على تعريف القيم الافتراضية للخاصيات. يمكن حل هذه المشكلة بواسطة التعليمة العامة للمصرف `withDefaults`:

```ts
export interface Props {
  msg?: string
  labels?: string[]
}

const props = withDefaults(defineProps<Props>(), {
  msg: 'hello',
  labels: () => ['one', 'two']
})
```

هذا سيصرف إلى خيارات زمن التشغيل المعادلة للخيارات الافتراضية. بالإضافة إلى ذلك، توفر الدالة المساعدة `withDefaults` التحقق من الأنواع للقيم الافتراضية، وتضمن أن نوع الخاصيات المُرجع يحتوي على علامات اختيارية مزيلة للخاصيات التي لها قيم افتراضية مصرح بها.

### بدون `<script setup>` {#without-script-setup}

إذا كنت لا تستخدم `<script setup>`، فإنه من الضروري استخدام `()defineComponent` لتمكين استنباط نوع الخاصيات. يُستنبط نوع كائن الخاصيات المُمرر إلى `()setup` من الوسيط `props`.

```ts
import { defineComponent } from 'vue'

export default defineComponent({
  props: {
    message: String
  },
  setup(props) {
    props.message // <-- type: string
  }
})
```

### الأنواع المعقدة للخاصيات {#complex-prop-types}

باستخدام التصريح على أساس الأنواع، يمكن للخاصية استخدام نوع معقد كما هو الحال بالنسبة لأي نوع آخر:

```vue
<script setup lang="ts">
interface Book {
  title: string
  author: string
  year: number
}

const props = defineProps<{
  book: Book
}>()
</script>
```

بالنسبة للتصريح التشغيلي، يمكننا استخدام نوع `PropType`:

```ts
import type { PropType } from 'vue'

const props = defineProps({
  book: Object as PropType<Book>
})
```

يعمل هذا بنفس الطريقة كما لو كنا نحدد خيار الخاصيات مباشرة:

```ts
import { defineComponent } from 'vue'
import type { PropType } from 'vue'

export default defineComponent({
  props: {
    book: Object as PropType<Book>
  }
})
```

يُستخدم خيار الخاصيات `props` بشكل أكثر شيوعًا مع واجهة الخيارات ، لذا ستجد أمثلة أكثر تفصيلاً في دليل [TypeScript مع واجهة الخيارات ](/guide/typescript/options-api#typing-component-props). تنطبق التقنيات الموضحة في تلك الأمثلة أيضًا على التصريحات التشغيلية باستخدام `()defineProps`.

## تحديد نوع إرسالات المكون {#typing-component-emits}

في `<script setup>`، يمكن تحديد نوع دالة `emit` باستخدام إما التصريح التشغيلي أو التصريح على أساس الأنواع:

```vue
<script setup lang="ts">
// زمن التشغيل
const emit = defineEmits(['change', 'update'])

//  معتمدة على الخيارات
const emit = defineEmits({
  change: (id: number) => {
    // تعيد `true` أو `false` للإشارة إلى
    // نجاح / فشل التحقق
  },
  update: (value: string) => {
    // تعيد `true` أو `false` للإشارة إلى
    // نجاح / فشل التحقق
  }
})

// على أساس الأنواع
const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
}>()

// 3.3+: alternative, more succinct syntax
const emit = defineEmits<{
  change: [id: number]
  update: [value: string]
}>()
</script>
```

نوع الوسيط يمكن أن يكون مما يلي:

1. نوع دالة قابلة للاستدعاء، ولكن مكتوبة كنوع حرفي مع [توقيعات الاستدعاء](https://www.typescriptlang.org/docs/handbook/2/functions.html#call-signatures). سيستخدم كنوع الدالة `emit` المُرجعة.
2. نوع حرفي حيث تكون المفاتيح هي أسماء الأحداث ، والقيم هي أنواع مصفوفة / صف تمثل المعاملات المقبولة الإضافية للحدث. يستخدم المثال أعلاه الصفوف المسماة حتى يمكن لكل وسيط أن يكون له اسم صريح.

كما نرى، فإن تصريح النوع يمنحنا تحكمًا أدق في قيود النوع للأحداث المرسلة.

عند عدم استخدام `<script setup>`، يمكن لـ `()defineComponent` استنباط الأحداث المسموح بها لدالة `emit` المُعرضة على سياق الإعداد:

```ts
import { defineComponent } from 'vue'

export default defineComponent({
  emits: ['change'],
  setup(props, { emit }) {
    emit('change') // <-- التحقق من النوع/ التكملة الآلية
  }
})
```

## تحديد نوع `()ref` {#typing-ref}

يُستنبط نوع المرجع التفاعلي (ref) من القيمة الأولية: 

```ts
import { ref } from 'vue'

// النوع المستنبط: Ref<number>
const year = ref(2020)

// => خطأ TS: لا يمكن تعيين نوع 'string' إلى نوع 'number'.
year.value = '2020'
```

في بعض الأحيان ، قد نحتاج إلى تحديد أنواع معقدة لقيمة المرجع الداخلية. يمكننا القيام بذلك باستخدام نوع `Ref`:

```ts
import { ref } from 'vue'
import type { Ref } from 'vue'

const year: Ref<string | number> = ref('2020')

year.value = 2020 // صحيح!
```

أو ، عن طريق تمرير وسيط نوعي عند استدعاء `ref()` لتجاوز الاستنباط الافتراضي:

```ts
// النوع الناتج: Ref<string | number>
const year = ref<string | number>('2020')

year.value = 2020 // صحيح!
```

إذا قمت بتحديد وسيط نوعي عام لكن تجاهلت القيمة الأولية ، فسيكون النوع الناتج نوعا متحدا يتضمن `undefined`:

```ts
// النوع المستنبط: Ref<number | undefined>
const n = ref<number>()
```

## تحديد نوع `()reactive` {#typing-reactive}

يستنبط `()reactive` أيضا النوع من الوسيط النوعي الخاص به:

```ts
import { reactive } from 'vue'

// النوع المستنبط: { title: string }
const book = reactive({ title: 'Vue 3 Guide' })
```

لتحديد نوع خاصية `reactive` ، يمكننا استخدام واجهات:

```ts
import { reactive } from 'vue'

interface Book {
  title: string
  year?: number
}

const book: Book = reactive({ title: 'Vue 3 Guide' })
```

:::tip نصيحة
لا يُوصى باستخدام وسيط نوعي عام لـ `()reactive` لأن النوع المُرجع ، والذي يتعامل مع فك المرجع التفاعلي المتداخل ، هو مختلف عن نوع وسيط النوع المعمم.
:::

## تحديد نوع `()computed` {#typing-computed}

يستنبط `()computed` نوعه بناءً على قيمة الإرجاع للمُحصل:

```ts
import { ref, computed } from 'vue'

const count = ref(0)

// النوع المستنبط: ComputedRef<number>
const double = computed(() => count.value * 2)

// => خطأ TS: لا يوجد خاصية 'split' على النوع 'number'.
const result = double.value.split('')
```

يمكنك أيضًا تحديد نوع محدد عبر وسيط نوعي معمم:

```ts
const double = computed<number>(() => {
  // خطأ في النوع إذا لم يرجع هذا الرقم
})
```

## تحديد نوع معالجات الأحداث {#typing-event-handlers}

عند التعامل مع أحداث DOM الأصلية ، قد يكون من المفيد تحديد الوسيط النوعي الذي نمرره إلى المعالج بشكل صحيح. دعونا نلقي نظرة على هذا المثال:

```vue
<script setup lang="ts">
function handleChange(event) {
  // `event` يحتوي على نوع `any` بشكل ضمني
  console.log(event.target.value)
}
</script>

<template>
  <input type="text" @change="handleChange" />
</template>
```

بدون توصيفات النوع ، سيكون وسيط `event` ضمنيًا لديه نوع `any`. سيؤدي هذا أيضًا إلى خطأ TS إذا كان `"strict": true` أو `"noImplicitAny": true` التي تستخدامهما في `tsconfig.json`. لذلك ، يُوصى بتوصيف النوع الخاص بوسيط معالجات الأحداث بشكل صريح. بالإضافة إلى ذلك ، قد تحتاج إلى تحويل صريح للخصائص على `event`:

```ts
function handleChange(event: Event) {
  console.log((event.target as HTMLInputElement).value)
}
```

## تحديد النوع لدوال التزويد و الحقن {#typing-provide-inject}

 عادة تجرى عملية التزويد والحقن في مكونات مستقلة. لتحديد النوع الصحيح للقيم المحقونة ، توفر Vue واجهة `InjectionKey` ، وهي نوع معمم يستند على `Symbol`. يمكن استخدامه لمزامنة نوع القيمة المحقونة بين المزود والمستهلك:

```ts
import { provide, inject } from 'vue'
import type { InjectionKey } from 'vue'

const key = Symbol() as InjectionKey<string>

provide(key, 'foo') //  تزويد قيمة غير سلسلة نصية ستؤدي إلى خطأ
const foo = inject(key) // نوع foo: string | undefined
```

يُوصى بوضع مفتاح الحقن في ملف منفصل حتى يمكن استيراده في مكونات متعددة.

عند استخدام مفاتيح الحقن النصية ، سيكون نوع القيمة المحقونة `unknown` ، ويحتاج إلى تعريف صريح عبر وسيط نوعي معمم:

```ts
const foo = inject<string>('foo') // type: string | undefined
```

لاحظ أن القيمة المحقونة لا يزال ممكنًا أن تكون `undefined` ، لأنه لا يوجد ضمان بأن مزود سيقدم هذه القيمة في وقت التشغيل.

يمكن إزالة نوع `undefined` عن طريق تزويد قيمة افتراضية:

```ts
const foo = inject<string>('foo', 'bar') // type: string
```

إذا كنت متأكدًا من أن القيمة مزودة دائمًا ، يمكنك أيضًا تحويل القيمة:

```ts
const foo = inject('foo') as string
```

## إضافة الأنواع إلى مراجع القالب{#typing-template-refs}

يجب إنشاء مراجع القالب بوسيط نوعي معمم صريح وقيمة افتراضية `null`:


```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

const el = ref<HTMLInputElement | null>(null)

onMounted(() => {
  el.value?.focus()
})
</script>

<template>
  <input ref="el" />
</template>
```

لاحظ أنه للحصول على أمان النوع الصارم ، فإنه من الضروري استخدام السلسلة الاختيارية أو مراقبي النوع عند الوصول إلى `el.value`. هذا لأن قيمة المرجع الافتراضية هي `null` حتى يوصل المكون ، ويمكن أيضًا تعيينها على `null` إذا فُصل المكون المرجعي من خلال `v-if`.

## تحديد نوع مراجع القالب للمكون {#typing-component-template-refs}

في بعض الأحيان ، قد تحتاج إلى توصيف مرجع قالب لمكون ابن لاستدعاء تابع عام له. على سبيل المثال ، لدينا مكون ابن `MyModal` مع تابع لفتح النافذة المنبثقة:

```vue
<!-- MyModal.vue -->
<script setup lang="ts">
import { ref } from 'vue'

const isContentShown = ref(false)
const open = () => (isContentShown.value = true)

defineExpose({
  open
})
</script>
```

للحصول على نوع المثيل من `MyModal` ، نحتاج أولاً  الحصول على نوعه عبر `typeof` ، ثم استخدام أداة `InstanceType` الخاصة بـ TypeScript لاستخراج نوع المثيل:

```vue{5}
<!-- App.vue -->
<script setup lang="ts">
import MyModal from './MyModal.vue'

const modal = ref<InstanceType<typeof MyModal> | null>(null)

const openModal = () => {
  modal.value?.open()
}
</script>
```

لاحظ أنه إذا كنت تريد استخدام هذه التقنية في ملفات TypeScript بدلاً من المكونات أحادية الملف لـVue  ، فيجب عليك تمكين [وضع الاستحواذ](./overview#volar-takeover-mode) لـ Volar.

في حالة عدم توفر النوع الدقيق للمكون أو عدم أهميته ، يمكن استخدام `ComponentPublicInstance` بدلاً من ذلك. سيستخدم هذا النوع فقط للخاصيات المشتركة بين جميع المكونات ، مثل `el$`:

```ts
import { ref } from 'vue'
import type { ComponentPublicInstance } from 'vue'

const child = ref<ComponentPublicInstance | null>(null)
```
