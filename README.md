# React + TypeScript + Vite + Zustand + TailwindCSS + ReactRouterDom

<img src="https://github.com/Klerith/zustand-mini-curso/blob/main/public/screenshot.png?raw=true" alt="Dashboard Screenshot">

## Instalar

1. Clonar proyecto `git clone https://github.com/Klerith/zustand-mini-curso.git`
2. Instalar dependencias `npm install`
3. Correr en desarrollo `npm run dev`

## Implementar zustand

1. instalar zustand

```
npm install zustand
```

2. (con javascript) crear store /src/stores/bears/bears.store.js

```
import { create } from 'zustand'

const useBearsStore = create((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
  removeAllBears: () => set({ bears: 0 }),
  updateBears: (newBears) => set({ bears: newBears }),
}))
```

2. (con typescript) crear store /src/stores/bears/bears.store.ts

```
import { create } from "zustand";

interface BearState {
 blackBears: number;
 polarBears: number;
 pandaBears: number;

 increaseBlackBears: (by: number) => void
}

export const useBearsStore = create<BearState>()((set) => ({
 blackBears: 10,
 polarBears: 5,
 pandaBears: 1,

 increaseBlackBears: (by: number) => set((state) => ({ blackBears: state.blackBears + by })),
}));
```

3. usar metodos y propiedades en los componentes

```
export const BlackBears = () => .....
 const blackBears = useBearsStore((state) => state.blackBears);

 const increaseBlackBears = useBearsStore((state) => state.increaseBlackBears);
```

## Objetos anidados en el store

1. /src/store/bears/bears.store.ts, en este ejemplo creamos un arreglo de objetos bear y una funcion que simplemente retorna el mismo estado, para mostrar como podemos evitar re-renders cuando las propiedades de un objeto no han cambiado en absoluto

```
import { create } from "zustand";

interface Bear {
 id: number;
 name: string;
}

interface BearState {
 blackBears: number;
 polarBears: number;
 pandaBears: number;

 bears: Bear[];

 increaseBlackBears: (by: number) => void;
 increasePolarBears: (by: number) => void;
 increasePandaBears: (by: number) => void;

 doNothing: () => void;
}

export const useBearsStore = create<BearState>()((set) => ({
 blackBears: 10,
 polarBears: 5,
 pandaBears: 1,

 bears: [{ id: 1, name: "oso #1" }],

 increaseBlackBears: (by: number) =>
  set((state) => ({ blackBears: state.blackBears + by })),
 increasePolarBears: (by: number) =>
  set((state) => ({ polarBears: state.polarBears + by })),
 increasePandaBears: (by: number) =>
  set((state) => ({ pandaBears: state.pandaBears + by })),

 doNothing: () => set((state) => ({ bears: [...state.bears] })),
}));
```

2. en el componente, usamos useShallow, detecta si hubo cambios en las propiedades de nuestro arreglo de osos evitando re-renders innecesarios

```
export const BearsDisplay = () => {
 const bears = useBearsStore(useShallow((state) => state.bears));

 const doNothing = useBearsStore((state) => state.doNothing);

 return (
  <WhiteCard centered>
   <h2>Osos</h2>

   <button onClick={() => doNothing()}> Do nothing </button>
   <pre>{JSON.stringify(bears, null, 2)}</pre>
  </WhiteCard>
 );
};
```

## Agregar o eliminar osos

```
import { create } from "zustand";

interface Bear {
 id: number;
 name: string;
}

interface BearState {

 bears: Bear[];

 doNothing: () => void;
 addBear: () => void;
 deleteBears: () => void;
}

export const useBearsStore = create<BearState>()((set) => ({

 bears: [{ id: 1, name: "oso #1" }],

 doNothing: () => set((state) => ({ bears: [...state.bears] })),

 // metodos para agregar un oso o eliminar la lista de osos
 addBear: () =>
  set((state) => ({
   bears: [
    ...state.bears,
    { id: state.bears.length + 1, name: `Oso #${state.bears.length + 1}` },
   ],
  })),
  // no necesitamos el state para eliminar todos los osos
 deleteBears: () => set({ bears: [] }),
}));
```

## Uso de getters

1. bears.store.ts, creamos una propiedad computada para obtener el total de osos, sumando nuestras propiedades

```
import { create } from "zustand";

interface Bear {
 id: number;
 name: string;
}

interface BearState {
 blackBears: number;
 polarBears: number;
 pandaBears: number;

 bears: Bear[];

// tipado para nuestra propiedad computada
 computed: {
  totalBears: number;
 };

 increaseBlackBears: (by: number) => void;
 increasePolarBears: (by: number) => void;
 increasePandaBears: (by: number) => void;

 doNothing: () => void;
 addBear: () => void;
 deleteBears: () => void;
}

// hacemos uso de get
export const useBearsStore = create<BearState>()((set, get) => ({
 blackBears: 10,
 polarBears: 5,
 pandaBears: 1,

 bears: [{ id: 1, name: "oso #1" }],

// creamos la propiedad computada que suma la cantidad total de osos
 computed: {
  get totalBears() {
   return (
    get().blackBears + get().polarBears + get().pandaBears + get().bears.length
   );
  },
 },

 increaseBlackBears: (by: number) =>
  set((state) => ({ blackBears: state.blackBears + by })),
 increasePolarBears: (by: number) =>
  set((state) => ({ polarBears: state.polarBears + by })),
 increasePandaBears: (by: number) =>
  set((state) => ({ pandaBears: state.pandaBears + by })),

 doNothing: () => set((state) => ({ bears: [...state.bears] })),
 addBear: () =>
  set((state) => ({
   bears: [
    ...state.bears,
    { id: state.bears.length + 1, name: `Oso #${state.bears.length + 1}` },
   ],
  })),
 deleteBears: () => set({ bears: [] }),
}));
```

2. usamos en nuestro componente la propiedad computada para mostrar la cantidad total de osos

```
export const Dashboard = () => ......
 const totalBears = useBearsStore((state) => state.computed.totalBears);

 return ......

     <p>{totalBears}</p> ....
```
