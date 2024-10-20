<script lang="ts">
  import { onMount } from "svelte";
  import { spring } from "svelte/motion";
  import { fade, fly } from "svelte/transition";

  interface Subsection {
    title: string;
    id: string;
  }

  interface Section {
    title: string;
    subsections: Subsection[];
  }

  const sections: Section[] = [
    {
      title: "Number Theory",
      subsections: [
        { title: "Prime Numbers", id: "prime-numbers" },
        { title: "Congruences", id: "congruences" },
        { title: "Diophantine Equations", id: "diophantine-equations" },
      ],
    },
    {
      title: "Abstract Algebra",
      subsections: [
        { title: "Group Theory", id: "group-theory" },
        { title: "Ring Theory", id: "ring-theory" },
        { title: "Galois Theory", id: "galois-theory" },
      ],
    },
    {
      title: "Analysis",
      subsections: [
        { title: "Real Analysis", id: "real-analysis" },
        { title: "Complex Analysis", id: "complex-analysis" },
        { title: "Functional Analysis", id: "functional-analysis" },
      ],
    },
    {
      title: "Topology",
      subsections: [
        { title: "Point-Set Topology", id: "point-set-topology" },
        { title: "Algebraic Topology", id: "algebraic-topology" },
      ],
    },
    {
      title: "Probability Theory",
      subsections: [
        { title: "Measure Theory", id: "measure-theory" },
        { title: "Stochastic Processes", id: "stochastic-processes" },
      ],
    },
  ];

  let activeSection = sections[0];
  let activeSubsections: string[] = [];
  let searchQuery = "";

  let sidebarHighlight = spring(
    { y: 0, height: 0 },
    {
      stiffness: 0.08,
      damping: 0.45,
    }
  );

  let sidebarItems: HTMLElement[] = [];

  function updateSidebarHighlight(entries: IntersectionObserverEntry[]) {
    let visibleSubsections = entries
      .filter((entry) => entry.isIntersecting)
      .map((entry) => entry.target.id);

    activeSubsections = visibleSubsections;

    if (visibleSubsections.length > 0) {
      const firstItem = sidebarItems.find(
        (item) => item.dataset.id === visibleSubsections[0]
      );
      const lastItem = sidebarItems.find(
        (item) =>
          item.dataset.id === visibleSubsections[visibleSubsections.length - 1]
      );

      if (firstItem && lastItem) {
        const firstRect = firstItem.getBoundingClientRect();
        const lastRect = lastItem.getBoundingClientRect();
        sidebarHighlight.set({
          y: firstRect.top,
          height: lastRect.bottom - firstRect.top,
        });
      }
    }

    const sectionIndex = sections.findIndex((s) =>
      s.subsections.some((sub) => visibleSubsections.includes(sub.id))
    );
    if (sectionIndex !== -1) {
      activeSection = sections[sectionIndex];
    }
  }

  onMount(() => {
    const observer = new IntersectionObserver(updateSidebarHighlight, {
      rootMargin: "-20% 0px -79% 0px",
      threshold: [0, 0.2, 0.4, 0.6, 0.8, 1],
    });

    document
      .querySelectorAll("section")
      .forEach((section) => observer.observe(section));

    return () => observer.disconnect();
  });

  function scrollToSection(sectionId: string) {
    const element = document.getElementById(sectionId);
    if (element) {
      element.scrollIntoView({ behavior: "smooth" });
    }
  }

  function nextSection() {
    const currentIndex = sections.indexOf(activeSection);
    if (currentIndex < sections.length - 1) {
      scrollToSection(sections[currentIndex + 1].subsections[0].id);
    }
  }
</script>

<div class="flex h-screen bg-black text-white overflow-hidden font-sans">
  <nav
    class="w-72 bg-gray-900 bg-opacity-50 backdrop-filter backdrop-blur-lg p-6 relative overflow-y-auto border-r border-gray-800"
  >
    <div class="text-xl font-bold mb-8 text-red-500 flex items-center">
      <svg
        class="w-6 h-6 mr-2"
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path
          d="M12 22C17.5228 22 22 17.5228 22 12C22 6.47715 17.5228 2 12 2C6.47715 2 2 6.47715 2 12C2 17.5228 6.47715 22 12 22Z"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
        />
        <path
          d="M12 18C15.3137 18 18 15.3137 18 12C18 8.68629 15.3137 6 12 6C8.68629 6 6 8.68629 6 12C6 15.3137 8.68629 18 12 18Z"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
        />
        <path
          d="M12 14C13.1046 14 14 13.1046 14 12C14 10.8954 13.1046 10 12 10C10.8954 10 10 10.8954 10 12C10 13.1046 10.8954 14 12 14Z"
          fill="currentColor"
        />
      </svg>
      Marix
    </div>
    <div
      class="mb-4 text-xs font-medium text-gray-400 uppercase tracking-wider"
    >
      Topics
    </div>
    {#each sections as section}
      <div class="mb-4">
        <div class="py-2 text-sm font-medium">{section.title}</div>
        {#each section.subsections as subsection}
          <div
            bind:this={sidebarItems[sidebarItems.length]}
            data-id={subsection.id}
            class="py-1 pl-4 text-xs cursor-pointer hover:text-red-400 transition-colors duration-150"
            class:text-red-500={activeSubsections.includes(subsection.id)}
            on:click={() => scrollToSection(subsection.id)}
          >
            {subsection.title}
          </div>
        {/each}
      </div>
    {/each}
    <div
      class="absolute left-0 w-0.5 bg-red-500 rounded-full transition-all duration-300 ease-out"
      style="top: {$sidebarHighlight.y}px; height: {$sidebarHighlight.height}px;"
    ></div>
  </nav>
  <main class="flex-1 p-8 overflow-y-auto bg-gray-900">
    <div class="max-w-3xl mx-auto">
      <div class="flex items-center justify-between mb-8">
        <div class="relative w-96">
          <input
            bind:value={searchQuery}
            placeholder="Search topics..."
            class="w-full bg-gray-800 bg-opacity-50 backdrop-filter backdrop-blur-lg text-white rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-red-500"
          />
          <svg
            class="w-5 h-5 text-gray-400 absolute right-3 top-2.5"
            fill="none"
            stroke="currentColor"
            viewBox="0 0 24 24"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
            ></path>
          </svg>
        </div>
        <div class="flex items-center space-x-6 text-sm">
          <button class="text-gray-300 hover:text-red-400 transition-colors"
            >Home</button
          >
          <button class="text-gray-300 hover:text-red-400 transition-colors"
            >PerRed</button
          >
          <button class="text-gray-300 hover:text-red-400 transition-colors"
            >Support</button
          >
          <button
            class="bg-red-500 text-white px-4 py-2 rounded-md text-sm font-medium hover:bg-red-600 transition-colors"
            >Sign in</button
          >
        </div>
      </div>
      {#each sections as section}
        <div class="mb-12">
          <h1 class="text-3xl font-bold mb-4 text-white">{section.title}</h1>
          {#each section.subsections as subsection}
            <section id={subsection.id} class="mb-8">
              <h2 class="text-2xl font-semibold mb-4 text-gray-200">
                {subsection.title}
              </h2>
              <p class="text-gray-300 mb-6 leading-relaxed">
                Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do
                eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut
                enim ad minim veniam, quis nostrud exercitation ullamco laboris
                nisi ut aliquip ex ea commodo consequat.
              </p>
              <div
                class="bg-gray-800 bg-opacity-50 backdrop-filter backdrop-blur-lg rounded-lg p-6 border border-gray-700"
              >
                <h3 class="text-xl font-semibold mb-4 text-gray-100">
                  Key Concepts
                </h3>
                <ul class="list-disc list-inside mb-6 text-gray-300">
                  <li>Fundamental principles of {subsection.title}</li>
                  <li>Important theorems and their applications</li>
                  <li>Advanced techniques and methodologies</li>
                </ul>
                <button class="text-red-400 hover:text-red-300 font-medium"
                  >Learn more about {subsection.title} →</button
                >
              </div>
            </section>
          {/each}
        </div>
      {/each}
      <div class="mt-12 flex justify-end">
        <button
          class="px-6 py-2 bg-red-500 text-white rounded-md hover:bg-red-600 transition-colors focus:outline-none focus:ring-2 focus:ring-red-400 focus:ring-offset-2 focus:ring-offset-gray-900"
          on:click={nextSection}
        >
          Next Section
        </button>
      </div>
    </div>
  </main>
</div>

<style>
  :global(body) {
    background-color: #000000;
    color: #ffffff;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
      "Helvetica Neue", Arial, sans-serif;
  }
</style>
