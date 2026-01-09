# Form Handling & Validation Standards

## 📝 Form Handling & Validation

### Core Libraries

- **React Hook Form** - Form state management dan validation
- **Yup** - Schema validation library
- **@hookform/resolvers** - Resolver untuk React Hook Form + Yup

### 1. Basic Form Setup

**Form Component dengan React Hook Form + Yup**

```typescript
"use client";

import { useForm } from "react-hook-form";
import { yupResolver } from "@hookform/resolvers/yup";
import * as yup from "yup";
import { Button } from "@/components/atoms/Button";
import { Input } from "@/components/atoms/Input";

// 1. Define Yup Schema
const loginSchema = yup.object({
  email: yup.string().email("Email tidak valid").required("Email wajib diisi"),
  password: yup
    .string()
    .min(8, "Password minimal 8 karakter")
    .required("Password wajib diisi"),
});

type LoginFormData = yup.InferType<typeof loginSchema>;

export default function LoginForm() {
  // 2. Setup React Hook Form dengan Yup resolver
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<LoginFormData>({
    resolver: yupResolver(loginSchema),
    defaultValues: {
      email: "",
      password: "",
    },
  });

  // 3. Handle Form Submission
  const onSubmit = async (data: LoginFormData) => {
    try {
      // API call atau Server Action
      await loginUser(data);
    } catch (error) {
      // Error handling
      console.error(error);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <Input
          {...register("email")}
          type="email"
          placeholder="Email"
          aria-invalid={!!errors.email}
        />
        {errors.email && (
          <p className="text-sm text-red-500 mt-1">{errors.email.message}</p>
        )}
      </div>

      <div>
        <Input
          {...register("password")}
          type="password"
          placeholder="Password"
          aria-invalid={!!errors.password}
        />
        {errors.password && (
          <p className="text-sm text-red-500 mt-1">{errors.password.message}</p>
        )}
      </div>

      <Button type="submit" disabled={isSubmitting}>
        {isSubmitting ? "Loading..." : "Login"}
      </Button>
    </form>
  );
}
```

### 2. Form dengan Server Actions

**Server Action Integration**

```typescript
"use client";

import { useForm } from "react-hook-form";
import { yupResolver } from "@hookform/resolvers/yup";
import * as yup from "yup";
import { useActionState } from "react";
import { createUser } from "@/actions/userActions";

const signupSchema = yup.object({
  name: yup.string().required("Nama wajib diisi"),
  email: yup.string().email().required("Email wajib diisi"),
  password: yup.string().min(8).required("Password wajib diisi"),
});

type SignupFormData = yup.InferType<typeof signupSchema>;

export default function SignupForm() {
  const [state, formAction, isPending] = useActionState(createUser, {
    message: "",
    errors: {},
  });

  const {
    register,
    handleSubmit,
    formState: { errors },
    setError,
  } = useForm<SignupFormData>({
    resolver: yupResolver(signupSchema),
  });

  // Sync server errors ke React Hook Form
  React.useEffect(() => {
    if (state.errors) {
      Object.entries(state.errors).forEach(([field, message]) => {
        setError(field as keyof SignupFormData, {
          type: "server",
          message: message as string,
        });
      });
    }
  }, [state.errors, setError]);

  const onSubmit = (data: SignupFormData) => {
    const formData = new FormData();
    formData.append("name", data.name);
    formData.append("email", data.email);
    formData.append("password", data.password);
    formAction(formData);
  };

  return <form onSubmit={handleSubmit(onSubmit)}>{/* Form fields */}</form>;
}
```

### 3. Form Field Component Pattern

**Reusable FormField Component**

```typescript
"use client";

import { useFormContext, Controller } from "react-hook-form";
import { Input } from "@/components/atoms/Input";
import { Label } from "@/components/atoms/Label";

interface FormFieldProps {
  name: string;
  label: string;
  type?: string;
  placeholder?: string;
  required?: boolean;
}

export function FormField({
  name,
  label,
  type = "text",
  placeholder,
  required = false,
}: FormFieldProps) {
  const {
    control,
    formState: { errors },
  } = useFormContext();

  return (
    <div className="space-y-2">
      <Label htmlFor={name}>
        {label}
        {required && <span className="text-red-500">*</span>}
      </Label>
      <Controller
        name={name}
        control={control}
        render={({ field }) => (
          <Input
            {...field}
            id={name}
            type={type}
            placeholder={placeholder}
            aria-invalid={!!errors[name]}
            aria-describedby={errors[name] ? `${name}-error` : undefined}
          />
        )}
      />
      {errors[name] && (
        <p id={`${name}-error`} className="text-sm text-red-500">
          {errors[name]?.message as string}
        </p>
      )}
    </div>
  );
}
```

**Usage dengan FormProvider**

```typescript
"use client";

import { FormProvider, useForm } from "react-hook-form";
import { yupResolver } from "@hookform/resolvers/yup";
import { FormField } from "@/components/molecules/FormField";

export default function MyForm() {
  const methods = useForm({
    resolver: yupResolver(schema),
  });

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(onSubmit)}>
        <FormField name="email" label="Email" type="email" required />
        <FormField name="password" label="Password" type="password" required />
      </form>
    </FormProvider>
  );
}
```

### 4. Advanced Yup Validation

**Complex Validation Patterns**

```typescript
import * as yup from "yup";

// Conditional Validation
const profileSchema = yup.object({
  role: yup.string().required(),
  companyName: yup.string().when("role", {
    is: "business",
    then: (schema) => schema.required("Nama perusahaan wajib diisi"),
    otherwise: (schema) => schema.notRequired(),
  }),
});

// Array Validation
const productSchema = yup.object({
  name: yup.string().required(),
  tags: yup
    .array()
    .of(yup.string().required())
    .min(1, "Minimal 1 tag")
    .max(5, "Maksimal 5 tag"),
  variants: yup
    .array()
    .of(
      yup.object({
        name: yup.string().required(),
        price: yup.number().positive().required(),
      })
    )
    .min(1, "Minimal 1 variant"),
});

// Custom Validation
const passwordSchema = yup.object({
  password: yup
    .string()
    .required()
    .test(
      "password-strength",
      "Password harus mengandung huruf besar, huruf kecil, dan angka",
      (value) => {
        if (!value) return false;
        return /(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(value);
      }
    ),
  confirmPassword: yup
    .string()
    .required()
    .oneOf([yup.ref("password")], "Password tidak sama"),
});

// Date Validation
const eventSchema = yup.object({
  startDate: yup.date().required(),
  endDate: yup
    .date()
    .required()
    .min(yup.ref("startDate"), "Tanggal akhir harus setelah tanggal mulai"),
});

// Async Validation (untuk check availability)
const usernameSchema = yup.object({
  username: yup
    .string()
    .required()
    .test("username-available", "Username sudah digunakan", async (value) => {
      if (!value) return false;
      const isAvailable = await checkUsernameAvailability(value);
      return isAvailable;
    }),
});
```

### 5. File Upload Pattern

**File Upload dengan React Hook Form**

```typescript
"use client";

import { useForm, Controller } from "react-hook-form";
import { useDropzone } from "react-dropzone";
import { useState } from "react";

const fileSchema = yup.object({
  avatar: yup
    .mixed<File>()
    .required("File wajib diupload")
    .test("file-size", "File maksimal 2MB", (value) => {
      if (!value) return false;
      return value.size <= 2 * 1024 * 1024;
    })
    .test("file-type", "File harus berupa gambar", (value) => {
      if (!value) return false;
      return ["image/jpeg", "image/png", "image/webp"].includes(value.type);
    }),
});

type FileFormData = yup.InferType<typeof fileSchema>;

export default function FileUploadForm() {
  const [preview, setPreview] = useState<string | null>(null);

  const { control, handleSubmit, setValue, watch } = useForm<FileFormData>({
    resolver: yupResolver(fileSchema),
  });

  const file = watch("avatar");

  const onDrop = (acceptedFiles: File[]) => {
    const file = acceptedFiles[0];
    if (file) {
      setValue("avatar", file);
      // Create preview
      const reader = new FileReader();
      reader.onloadend = () => {
        setPreview(reader.result as string);
      };
      reader.readAsDataURL(file);
    }
  };

  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    onDrop,
    accept: {
      "image/*": [".jpeg", ".jpg", ".png", ".webp"],
    },
    multiple: false,
  });

  const onSubmit = async (data: FileFormData) => {
    const formData = new FormData();
    formData.append("avatar", data.avatar);
    // Upload ke server
    await uploadAvatar(formData);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Controller
        name="avatar"
        control={control}
        render={({ field, fieldState: { error } }) => (
          <div>
            <div
              {...getRootProps()}
              className={`border-2 border-dashed p-8 text-center cursor-pointer ${
                isDragActive ? "border-blue-500" : "border-gray-300"
              }`}
            >
              <input {...getInputProps()} />
              {preview ? (
                <img src={preview} alt="Preview" className="max-h-48 mx-auto" />
              ) : (
                <p>Drag & drop atau klik untuk upload</p>
              )}
            </div>
            {error && <p className="text-red-500">{error.message}</p>}
          </div>
        )}
      />
      <Button type="submit">Upload</Button>
    </form>
  );
}
```

### 6. Multi-Step Form Pattern

**Wizard Form dengan Multiple Steps**

```typescript
"use client";

import { useForm, FormProvider } from "react-hook-form";
import { useState } from "react";
import { Step1 } from "./Step1";
import { Step2 } from "./Step2";
import { Step3 } from "./Step3";

const multiStepSchema = yup.object({
  // Step 1
  name: yup.string().required(),
  email: yup.string().email().required(),
  // Step 2
  address: yup.string().required(),
  city: yup.string().required(),
  // Step 3
  paymentMethod: yup.string().required(),
});

type MultiStepFormData = yup.InferType<typeof multiStepSchema>;

const steps = [
  {
    id: 1,
    component: Step1,
    schema: yup.object({
      name: yup.string().required(),
      email: yup.string().email().required(),
    }),
  },
  {
    id: 2,
    component: Step2,
    schema: yup.object({
      address: yup.string().required(),
      city: yup.string().required(),
    }),
  },
  {
    id: 3,
    component: Step3,
    schema: yup.object({ paymentMethod: yup.string().required() }),
  },
];

export default function MultiStepForm() {
  const [currentStep, setCurrentStep] = useState(0);
  const CurrentStepComponent = steps[currentStep].component;

  const methods = useForm<MultiStepFormData>({
    mode: "onChange",
    defaultValues: {
      name: "",
      email: "",
      address: "",
      city: "",
      paymentMethod: "",
    },
  });

  const validateStep = async () => {
    const stepSchema = steps[currentStep].schema;
    const isValid = await methods.trigger(Object.keys(stepSchema.fields));
    return isValid;
  };

  const handleNext = async () => {
    const isValid = await validateStep();
    if (isValid && currentStep < steps.length - 1) {
      setCurrentStep(currentStep + 1);
    }
  };

  const handlePrevious = () => {
    if (currentStep > 0) {
      setCurrentStep(currentStep - 1);
    }
  };

  const onSubmit = async (data: MultiStepFormData) => {
    // Submit final data
    console.log("Final data:", data);
  };

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(onSubmit)}>
        <CurrentStepComponent />

        <div className="flex justify-between mt-8">
          <Button
            type="button"
            onClick={handlePrevious}
            disabled={currentStep === 0}
          >
            Previous
          </Button>

          {currentStep < steps.length - 1 ? (
            <Button type="button" onClick={handleNext}>
              Next
            </Button>
          ) : (
            <Button type="submit">Submit</Button>
          )}
        </div>
      </form>
    </FormProvider>
  );
}
```

### 7. Form dengan React Query Mutation

**Form Submission dengan Optimistic Updates**

```typescript
"use client";

import { useForm } from "react-hook-form";
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { createPost } from "@/api/postApi";
import { toast } from "sonner";

const postSchema = yup.object({
  title: yup.string().required("Judul wajib diisi"),
  content: yup.string().required("Konten wajib diisi"),
});

type PostFormData = yup.InferType<typeof postSchema>;

export default function PostForm() {
  const queryClient = useQueryClient();

  const {
    register,
    handleSubmit,
    reset,
    formState: { errors },
  } = useForm<PostFormData>({
    resolver: yupResolver(postSchema),
  });

  const mutation = useMutation({
    mutationFn: createPost,
    onSuccess: (data) => {
      // Invalidate queries
      queryClient.invalidateQueries({ queryKey: ["posts"] });
      // Reset form
      reset();
      toast.success("Post berhasil dibuat");
    },
    onError: (error) => {
      toast.error(error.message || "Gagal membuat post");
    },
  });

  const onSubmit = (data: PostFormData) => {
    mutation.mutate(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("title")} />
      {errors.title && <p>{errors.title.message}</p>}

      <textarea {...register("content")} />
      {errors.content && <p>{errors.content.message}</p>}

      <Button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? "Menyimpan..." : "Publish"}
      </Button>
    </form>
  );
}
```

## Summary Checklist

1. [ ] **Schema**: Define Yup schema dengan validasi yang sesuai
2. [ ] **Resolver**: Gunakan `yupResolver` untuk integrasi dengan React Hook Form
3. [ ] **Types**: Infer types dari schema menggunakan `yup.InferType`
4. [ ] **Register**: Register fields dengan `register` atau `Controller`
5. [ ] **Errors**: Handle dan display error messages dengan proper accessibility
6. [ ] **Server Actions**: Integrate dengan Server Actions jika menggunakan Server Components
7. [ ] **File Upload**: Gunakan `Controller` dengan dropzone untuk file uploads
8. [ ] **Multi-Step**: Implement validation per step untuk wizard forms
9. [ ] **Async Validation**: Use Yup `.test()` untuk async validation (username availability, etc.)
10. [ ] **Error Handling**: Handle server-side errors dan sync ke React Hook Form
