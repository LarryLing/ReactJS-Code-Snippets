<!-- Improved compatibility of back to top link: See: https://github.com/othneildrew/Best-README-Template/pull/73 -->

<a id="readme-top"></a>

<!--
*** Thanks for checking out the Best-README-Template. If you have a suggestion
*** that would make this better, please fork the repo and create a pull request
*** or simply open an issue with the tag "enhancement".
*** Don't forget to give the project a star!
*** Thanks again! Now go create something AMAZING! :D
-->

<!-- PROJECT SHIELDS -->
<!--
*** I'm using markdown "reference style" links for readability.
*** Reference links are enclosed in brackets [ ] instead of parentheses ( ).
*** See the bottom of this document for the declaration of the reference variables
*** for contributors-url, forks-url, etc. This is an optional, concise syntax you may use.
*** https://www.markdownguide.org/basic-syntax/#reference-style-links
-->

[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![Unlicense License][license-shield]][license-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/othneildrew/Best-README-Template">
    <img src="images/logo.png" alt="Logo" width="80" height="80">
  </a>

  <h3 align="center">NextJS + ShadCN/Tailwind + Supabase Template</h3>

  <p align="center">
    A template repository including basic navigation and authentication/authorization functionality.
    <br />
    <br />
    <a href="https://github.com/othneildrew/Best-README-Template">View Demo</a>
    &middot;
    <a href="https://github.com/othneildrew/Best-README-Template/issues/new?labels=bug&template=bug-report---.md">Report Bug</a>
    &middot;
    <a href="https://github.com/othneildrew/Best-README-Template/issues/new?labels=enhancement&template=feature-request---.md">Request Feature</a>
  </p>
</div>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>

<!-- ABOUT THE PROJECT -->

## About The Project

[![Product Name Screen Shot][product-screenshot]](https://example.com)

There are many great README templates available on GitHub; however, I didn't find one that really suited my needs so I created this enhanced one. I want to create a README template so amazing that it'll be the last one you ever need -- I think this is it.

Here's why:

- Your time should be focused on creating something amazing. A project that solves a problem and helps others
- You shouldn't be doing the same tasks over and over like creating a README from scratch
- You should implement DRY principles to the rest of your life :smile:

Of course, no one template will serve all projects since your needs may be different. So I'll be adding more in the near future. You may also suggest changes by forking this repo and creating a pull request or opening an issue. Thanks to all the people have contributed to expanding this template!

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Built With

- [![Next][Next.js]][Next-url]
- [![React][React.js]][React-url]
- [![Typescript][Typescript]][Typescript-url]
- [![Tailwind][TailwindCSS]][Tailwind-url]
- [![ShadCN][ShadCN]][ShadCN-url]
- [![RadixUI][RadixUI]][RadixUI-url]
- [![Supabase][Supabase]][Supabase-url]
- [![Postgres][Postgres]][Postgres-url]
- [![Vercel][Vercel]][Vercel-url]
- [![Zod][Zod]][Zod-url]

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- GETTING STARTED -->

## Getting Started

### Prerequisites

- npm
    ```sh
    npm install npm@latest -g
    ```

### Installation

1. Clone the repo
    ```sh
    git clone https://github.com/LarryLing/NextJS-Tailwind-Template.git
    ```
2. Install NPM packages
    ```sh
    npm install
    ```
3. Change git remote url to avoid accidental pushes to base project
    ```sh
    git remote set-url origin github_username/NextJS-Tailwind-Template
    git remote -v
    ```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- USAGE EXAMPLES -->

## Usage

### Connecting to Supabase

1. Create a `.env.local` file in the root directory with the following fields
    ```sh
    NEXT_PUBLIC_SUPABASE_URL=<SUPABASE_URL>
    NEXT_PUBLIC_SUPABASE_ANON_KEY=<SUPABASE_ANON_KEY>
    ```
2. Create a new Supabase project and replace `<YOUR_SUPABASE_URL>` and `<SUPABASE_ANON_KEY>`
   with the connection variables provided by Supabase

3. Make sure your `.gitignore` file contains `.env.local` before pushing to a public repository

### Managing users

1. Inside your Supabase project dashboard, navigate to the SQL Editor and run the following queries

    ```sh
    CREATE TABLE profiles (
        id UUID PRIMARY KEY DEFAULT auth.uid(),
        display_name TEXT NOT NULL,
        email TEXT NOT NULL DEFAULT auth.email() UNIQUE,
        avatar TEXT,
        role TEXT NOT NULL DEFAULT 'other',
        bio TEXT DEFAULT ''
    );

    ALTER TABLE profiles
        ADD CONSTRAINT fk_user
        FOREIGN KEY (id)
        REFERENCES auth.users (id)
        ON DELETE CASCADE;

    ALTER TABLE profiles
        ENABLE ROW LEVEL SECURITY;

    CREATE POLICY "User can only update own profile data"
    ON profiles
    FOR UPDATE
    TO public
    USING (
        (auth.uid() = id)
    );

    CREATE POLICY "User can only select own profile data"
    ON profiles
    FOR SELECT
    TO public
    USING (
        (auth.uid() = id)
    );
    ```

    ```sh
    CREATE FUNCTION public.handle_new_user()
    RETURNS TRIGGER
    LANGUAGE plpgsql
    SECURITY DEFINER SET search_path = public
    AS $$
    BEGIN
        INSERT INTO public.profiles(id, display_name, email)
        VALUES (NEW.id, NEW.raw_user_meta_data ->> 'display_name', NEW.email);
        RETURN NEW;
    END;
    $$;

    CREATE TRIGGER on_auth_user_created
    AFTER INSERT on auth.users
    FOR EACH ROW EXECUTE PROCEDURE public.handle_new_user();

    CREATE FUNCTION public.handle_update_user()
    RETURNS TRIGGER
    LANGUAGE plpgsql
    SECURITY DEFINER SET search_path = public
    AS $$
    BEGIN
        UPDATE public.profiles
        SET email = NEW.email
        WHERE id = NEW.id;
        RETURN NEW;
    END;
    $$;

    CREATE TRIGGER on_auth_user_updated
    AFTER UPDATE on auth.users
    FOR EACH ROW EXECUTE PROCEDURE public.handle_update_user();
    ```

    ```sh
    CREATE FUNCTION public.handle_password_change("current" text, "new" text, "userid" uuid)
    RETURNS TRIGGER
    LANGUAGE plpgsql
    SECURITY DEFINER SET search_path = public
    AS $$
    DECLARE encpass auth.users.encrypted_password%type;
    BEGIN
        SELECT encrypted_password
        FROM auth.users
        INTO encpass
        WHERE id = userid and encrypted_password = crypt(current, auth.users.encrypted_password);

        IF NOT FOUND THEN
            RETURN false;
        ELSE
            UPDATE auth.users SET encrypted_password = crypt(new, gen_salt('bf')) WHERE id = userid;
            RETURN true;
        END IF;
    END;
    $$;
    ```

### Email Templates

1. Navigate to the Email Templates subsection within the Authentication section of the project dashboard and enter the following for the `Confirm Signup` email.
    ```sh
    <h2>Confirm your signup</h2>
    <p>Follow this link to confirm your user:</p>
    <p><a href="{{ .SiteURL }}/auth/confirm?token_hash={{ .TokenHash }}&type=signup">Confirm your mail</a></p>
    ```

### Creating a storage bucket for avatars

1. Navigate to the Storage section in the Supabase project dashboard and create a new public bucket named `avatars`

2. Return to the SQL Editor section and run the following queries

    ```sh
    ALTER TABLE storage.objects ENABLE ROW LEVEL SECURITY;

    CREATE POLICY objects_select_policy ON storage.objects FOR SELECT
    USING (auth.role() = 'authenticated');

    CREATE POLICY objects_insert_policy ON storage.objects FOR INSERT
    WITH CHECK (auth.role() = 'authenticated');

    CREATE POLICY objects_update_policy ON storage.objects FOR UPDATE
    USING (auth.role() = 'authenticated');

    CREATE POLICY objects_delete_policy ON storage.objects FOR DELETE
    USING (auth.role() = 'authenticated');
    ```

### Settings up OAuth Third Party Providers

This template comes ready Discord and Github social authentication. If you want to add
additional social authenticators, please refer to the [Supabase documentation](https://supabase.com/docs/guides/auth/social-login)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- ROADMAP -->

## Roadmap

- [x] Add navbar popovers
    - [x] User popover
    - [x] Avatar popover
- [x] Add error handling
    - [x] Add error page(s)
    - [x] Redirect user to signup page with an error if email is taken
    - [x] Sign out page refresh
- [x] Add settings page template
    - [x] Create UI
    - [x] Fix backdrop bug
    - [x] Close other UI on dialog open
    - [x] Enable users to upload custom profile pictures
    - [x] Profile updates
    - [x] Password recovery
    - [x] Email updates
- [ ] Implement "Forgot Password" functionality
- [ ] Implement shadow for suspense rendering
- [ ] Update README with instructions for use

See the [open issues](https://github.com/othneildrew/Best-README-Template/issues) for a full list of proposed features (and known issues).

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- CONTRIBUTING -->

## Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

If you have a suggestion that would make this better, please fork the repo and create a pull request. You can also simply open an issue with the tag "enhancement".
Don't forget to give the project a star! Thanks again!

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- LICENSE -->

## License

Distributed under the MIT License. See `LICENSE.md` for more information.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- CONTACT -->

## Contact

Larry Ling - [LinkedIn](https://www.linkedin.com/in/larry-ling-student/) - larryling.main@gmail.com

Project Link: [https://github.com/LarryLing/NextJS-Tailwind-Template](https://github.com/LarryLing/NextJS-Tailwind-Template)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- ACKNOWLEDGMENTS -->

## Acknowledgments

- [Choose an Open Source License](https://choosealicense.com)
- [NextJS Docs](https://nextjs.org/docs)
- [TailwindCSS Docs](https://tailwindcss.com/)
- [Class Variance Authority Docs](https://cva.style/docs)
- [CLSX](https://github.com/lukeed/clsx)
- [Tailwind-Merge](https://github.com/dcastil/tailwind-merge)
- [CSS Flexbox Layout Guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [CSS Grid Layout Guide](https://css-tricks.com/snippets/css/complete%20%20-guide-grid/)
- [Lucide Icons](https://lucide.dev/)
- [Bootstrap Icons](https://icons.getbootstrap.com/)
- [Markdown Badges](https://github.com/Ileriayo/markdown-badges/blob/master/README.md#-design)
- [Badges 4 Markdown](https://github.com/alexandresanlim/Badges4-README.md-Profile)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- MARKDOWN LINKS & IMAGES -->

[contributors-shield]: https://img.shields.io/github/contributors/othneildrew/Best-README-Template.svg?style=for-the-badge
[contributors-url]: https://github.com/othneildrew/Best-README-Template/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/othneildrew/Best-README-Template.svg?style=for-the-badge
[forks-url]: https://github.com/othneildrew/Best-README-Template/network/members
[stars-shield]: https://img.shields.io/github/stars/othneildrew/Best-README-Template.svg?style=for-the-badge
[stars-url]: https://github.com/othneildrew/Best-README-Template/stargazers
[issues-shield]: https://img.shields.io/github/issues/othneildrew/Best-README-Template.svg?style=for-the-badge
[issues-url]: https://github.com/othneildrew/Best-README-Template/issues
[license-shield]: https://img.shields.io/github/license/othneildrew/Best-README-Template.svg?style=for-the-badge
[license-url]: https://github.com/othneildrew/Best-README-Template/blob/master/LICENSE.txt
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/in/othneildrew
[product-screenshot]: images/screenshot.png
[Next.js]: https://img.shields.io/badge/next.js-000000?style=for-the-badge&logo=nextdotjs&logoColor=white
[Next-url]: https://nextjs.org/
[React.js]: https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB
[React-url]: https://reactjs.org/
[Typescript]: https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white
[Typescript-url]: https://www.typescriptlang.org/
[TailwindCSS]: https://img.shields.io/badge/tailwindcss-%2338B2AC.svg?style=for-the-badge&logo=tailwind-css&logoColor=white
[Tailwind-url]: https://tailwindui.com/
[ShadCN]: https://img.shields.io/badge/shadcn%2Fui-000000?style=for-the-badge&logo=shadcnui&logoColor=white
[ShadCN-url]: https://ui.shadcn.com/
[RadixUI]: https://img.shields.io/badge/radix%20ui-161618.svg?style=for-the-badge&logo=radix-ui&logoColor=white
[RadixUI-url]: https://www.radix-ui.com/
[Supabase]: https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white
[Supabase-url]: https://supabase.com/
[Postgres]: https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white
[Postgres-url]: https://www.postgresql.org/
[Vercel]: https://img.shields.io/badge/vercel-%23000000.svg?style=for-the-badge&logo=vercel&logoColor=white
[Vercel-url]: https://vercel.com/
[Zod]: https://img.shields.io/badge/zod-%233068b7.svg?style=for-the-badge&logo=zod&logoColor=white
[Zod-url]: https://zod.dev/
