# How top companies write React JS

## Folder structure that scales

src 
├── features
│   └── auth
│      ├── components
│      ├── hooks
│      └── services
├── shared
│      ├── ui
│      └── utils
└── App.jsx

### Why
- Easier testing
- Better code ownership
- scales across large teams

## Design Systems and UI libraries

They don't write UI from scratch for every component


### Why
- Consistency 
- Faster UI Development 
- Accessibility built-in 

## Logic = Hook Always
Big companies extract buisness logic into custom hooks.
Example:
```JavaScript
//useProduct.js
export function useProducts() {
    const [data, setData] = useState([]);
    useEffect(() => {
        fetchProducts().then(setData);
    }, [])

    return data;
}
```

### Why
- Keep components clean
- Easy to test
- Reusable across the app

## Clean Code = Scalable Code
Big companies follow clean code principles like:
- SRP (Single Responsibility Principle)
- KISS (Keep It Simple, Stupid)
- DRY (Don't Repeat Yourself)

**Avoid**
```JavaScript
<button variant="danger" color="red" bg="red" />
```

**Do**
```JavaScript
<DeleteButton />
```

**Abstraction = fewer bugs**

## Real-time code reviews + Linting
They never push unreviewed code:
- Code reviews = learning + consistency 
- Pre-commit hooks using Husky 
- Code style enforced with ESList, Prettier

### Bonus Tip:
Try using commitlint, lint-staged, and conventional commits like top dev teams.

## Security & Accessibility First
Big companies care deeply about:
- Security
  - Preventing XSS, CSRF
  - Token handling
- Accessibility (ally)
  - Keyboard navigation
  - Screen reader support
  - Color contrast

**Use tools like:**
- eslint-plugin-jsx-ally
- axe-care
- Lighthouse

## Automation and monitoring
They automate everything:
- CI/CD (GitHub Actions, Vercel, CircleCI)
- Bundle size alerts 
- Lighthouse CI for performance

They don't just ship, they monitor what they ship 





> [!NOTE]
> Useful information that users should know, even when skimming content.

> [!TIP]
> Helpful advice for doing things better or more easily.

> [!IMPORTANT]
> Key information users need to know to achieve their goal.

> [!WARNING]
> Urgent info that needs immediate user attention to avoid problems.

> [!CAUTION]
> Advises about risks or negative outcomes of certain actions.