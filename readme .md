bloque dinámico para todas las tablas del esquema public, otorgando permiso total a todos para leer, insertar, actualizar y eliminar:

    sql

    DO $$
    DECLARE
      tabla TEXT;
      accion TEXT;
      politica TEXT;
    BEGIN
      FOR tabla IN SELECT tablename FROM pg_tables WHERE schemaname = 'public' LOOP

        -- Habilitar Row-Level Security
        EXECUTE format('ALTER TABLE public.%I ENABLE ROW LEVEL SECURITY;', tabla);

        -- Crear política de SELECT
        politica := 'select_all_' || tabla;
        IF NOT EXISTS (
          SELECT 1 FROM pg_policies WHERE schemaname = 'public' AND tablename = tabla AND policyname = politica
        ) THEN
          EXECUTE format('CREATE POLICY %I ON public.%I FOR SELECT USING (true);', politica, tabla);
        END IF;

        -- Crear política de INSERT
        politica := 'insert_all_' || tabla;
        IF NOT EXISTS (
          SELECT 1 FROM pg_policies WHERE schemaname = 'public' AND tablename = tabla AND policyname = politica
        ) THEN
          EXECUTE format('CREATE POLICY %I ON public.%I FOR INSERT WITH CHECK (true);', politica, tabla);
        END IF;

        -- Crear política de UPDATE
        politica := 'update_all_' || tabla;
        IF NOT EXISTS (
          SELECT 1 FROM pg_policies WHERE schemaname = 'public' AND tablename = tabla AND policyname = politica
        ) THEN
          EXECUTE format('CREATE POLICY %I ON public.%I FOR UPDATE USING (true);', politica, tabla);
        END IF;

        -- Crear política de DELETE
        politica := 'delete_all_' || tabla;
        IF NOT EXISTS (
          SELECT 1 FROM pg_policies WHERE schemaname = 'public' AND tablename = tabla AND policyname = politica
        ) THEN
          EXECUTE format('CREATE POLICY %I ON public.%I FOR DELETE USING (true);', politica, tabla);
        END IF;

      END LOOP;
    END $$;


🧠 Notas finales
    Esto aplica a todas las tablas actuales del esquema public.
    Si luego creas nuevas tablas, deberás re-ejecutar este bloque.
    Ideal solo para entornos de desarrollo o pruebas. En producción, deberías limitar el acceso por roles.


