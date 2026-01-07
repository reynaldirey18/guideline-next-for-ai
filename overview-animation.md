# Animation Patterns

## 🎨 Animation Patterns

### GSAP Animation

**Basic GSAP Animation**

```typescript
"use client";

import { gsap } from "gsap";
import { useEffect, useRef } from "react";

export default function AnimatedComponent() {
  const elementRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (elementRef.current) {
      gsap.to(elementRef.current, {
        duration: 1,
        x: 100,
        opacity: 1,
        ease: "power2.out",
      });
    }
  }, []);

  return (
    <div ref={elementRef} className="opacity-0">
      Animated
    </div>
  );
}
```

**GSAP Timeline**

```typescript
"use client";

import { gsap } from "gsap";
import { useEffect, useRef } from "react";

export default function TimelineAnimation() {
  const containerRef = useRef<HTMLDivElement>(null);
  const item1Ref = useRef<HTMLDivElement>(null);
  const item2Ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const tl = gsap.timeline({ repeat: -1, yoyo: true });

    tl.to(item1Ref.current, {
      duration: 1,
      x: 100,
      ease: "power1.inOut",
    }).to(
      item2Ref.current,
      {
        duration: 1,
        y: 50,
        ease: "power1.inOut",
      },
      "-=0.5" // Start 0.5s before previous animation ends
    );

    return () => {
      tl.kill(); // Cleanup
    };
  }, []);

  return (
    <div ref={containerRef}>
      <div ref={item1Ref}>Item 1</div>
      <div ref={item2Ref}>Item 2</div>
    </div>
  );
}
```

**SVG Path Animation dengan GSAP**

```typescript
"use client";

import { gsap } from "gsap";
import { useEffect, useRef } from "react";

export default function BlobAnimation() {
  const pathRef = useRef<SVGPathElement>(null);
  const paths = ["path1", "path2", "path3"]; // SVG path strings

  useEffect(() => {
    if (!pathRef.current) return;

    const animate = gsap.timeline({
      repeat: -1,
      yoyo: true,
      defaults: { ease: "sine.inOut" },
    });

    paths.forEach((path) => {
      animate.to(pathRef.current, {
        duration: 2,
        attr: { d: path },
      });
    });

    return () => {
      animate.kill();
    };
  }, []);

  return (
    <svg>
      <path ref={pathRef} d={paths[0]} />
    </svg>
  );
}
```

### Framer Motion Animation

**Basic Framer Motion**

```typescript
"use client";

import { motion } from "framer-motion";

export default function MotionComponent() {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.5 }}
    >
      Animated Content
    </motion.div>
  );
}
```

**Framer Motion dengan Variants**

```typescript
"use client";

import { motion } from "framer-motion";

const containerVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: {
      staggerChildren: 0.1,
    },
  },
};

const itemVariants = {
  hidden: { y: 20, opacity: 0 },
  visible: { y: 0, opacity: 1 },
};

export default function StaggeredAnimation() {
  return (
    <motion.div variants={containerVariants} initial="hidden" animate="visible">
      {[1, 2, 3].map((i) => (
        <motion.div key={i} variants={itemVariants}>
          Item {i}
        </motion.div>
      ))}
    </motion.div>
  );
}
```
