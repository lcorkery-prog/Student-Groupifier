# Student-Groupifier
Create balanced student groups.
When giving students a score for behavior, 1 is very good behavior and 5 is very poor behavior. When giving students a score for academics, A is very strong in your subject area and E is very weak in your subject area.

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Balanced Student Group Generator</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    <style>
        /* Custom styles for better focus state and feel */
        .card {
            @apply bg-white p-6 rounded-xl shadow-lg border border-gray-100;
        }
        .input-field {
            @apply w-full p-2 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500 transition duration-150;
        }
        /* ENHANCED PRIMARY BUTTON STYLING */
        .btn-primary {
            /* Base styles: brighter color, bolder text (font-extrabold), largest size (text-2xl), increased padding, rounded corners */
            @apply bg-blue-600 hover:bg-blue-700 active:bg-blue-800 text-white font-extrabold text-2xl py-4 px-8 rounded-xl transition duration-200 shadow-xl transform hover:scale-[1.01];
            /* Custom blue shadow for extra pop */
            box-shadow: 0 8px 20px -6px rgba(59, 130, 246, 0.7); /* Blue-600 glow */
        }
        .btn-secondary {
            @apply bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition duration-150;
        }
        .btn-export {
            @apply bg-green-500 hover:bg-green-600 text-white font-medium py-2 px-4 rounded-lg transition duration-150;
        }
        .btn-pdf { /* Renamed from btn-print for clarity */
            @apply bg-indigo-500 hover:bg-indigo-600 text-white font-medium py-2 px-4 rounded-lg transition duration-150;
        }
        .dropdown {
            @apply p-2 border border-gray-300 rounded-lg appearance-none bg-white pr-8;
        }

        /* Loading Spinner CSS */
        .spinner {
            border: 4px solid rgba(255, 255, 255, 0.3);
            border-top: 4px solid #fff;
            border-radius: 50%;
            width: 1.5rem;
            height: 1.5rem;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="bg-gray-50 min-h-screen p-4 sm:p-8 font-sans">

    <!-- Load PDF Generation Libraries -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

    <div id="app" class="max-w-7xl mx-auto">
        <h1 class="text-3xl font-extrabold text-gray-900 mb-8">Class Grouping Tool</h1>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
            <!-- Student Roster and Constraints Column (2/3 width on large screens) -->
            <div class="lg:col-span-2 space-y-8">
                
                <!-- Student Roster -->
                <div class="card">
                    <div class="flex justify-between items-center mb-4">
                        <h2 class="text-xl font-bold text-gray-800">Student Roster</h2>
                        <!-- Placeholder for Import from CSV - not implemented for simplicity -->
                        <button class="btn-secondary text-sm">Import from CSV (Mock)</button>
                    </div>

                    <div class="overflow-x-auto">
                        <table class="min-w-full divide-y divide-gray-200">
                            <thead>
                                <tr class="text-xs font-medium text-gray-500 uppercase tracking-wider bg-gray-50">
                                    <th class="px-3 py-3 text-left">Name</th>
                                    <th class="px-3 py-3 text-left">Behavior (1-5)</th>
                                    <th class="px-3 py-3 text-left">Academics (A-E)</th>
                                    <th class="px-3 py-3 text-left">Role</th>
                                    <th class="px-3 py-3 text-left">Actions</th>
                                </tr>
                            </thead>
                            <tbody id="student-list" class="bg-white divide-y divide-gray-200">
                                <!-- Student rows will be injected here -->
                            </tbody>
                        </table>
                    </div>

                    <div id="empty-roster-message" class="text-center py-4 text-gray-500 italic hidden">
                        No students added yet. Click "Add Student" to get started.
                    </div>
                    
                    <button onclick="addStudentRow()" class="mt-4 w-full border border-dashed border-gray-300 hover:border-blue-500 text-gray-600 hover:text-blue-600 py-3 rounded-lg transition duration-150 flex items-center justify-center">
                        <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                        Add Student
                    </button>
                </div>
                
                <!-- Constraints Section -->
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <!-- Must Be Together -->
                    <div class="card">
                        <!-- This heading remains text-xl -->
                        <h2 class="text-xl font-bold text-gray-800 mb-3">Must Be Together</h2>
                        <p class="text-sm text-gray-500 mb-4">Select students who must be in the same group.</p>
                        <div id="must-be-together-list" class="space-y-3">
                            <!-- Constraints go here -->
                        </div>
                        <button onclick="addConstraint('together')" class="mt-4 w-full border border-dashed border-gray-300 hover:border-green-500 text-gray-600 hover:text-green-600 py-2 rounded-lg transition duration-150 flex items-center justify-center">
                            <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                            Add Constraint
                        </button>
                    </div>

                    <!-- Must Be Separated -->
                    <div class="card">
                        <h2 class="text-xl font-bold text-gray-800 mb-3">Must Be Separated</h2>
                        <p class="text-sm text-gray-500 mb-4">Select students who must not be in the same group.</p>
                        <div id="conflict-constraints-list" class="space-y-3">
                            <!-- Conflicts go here -->
                        </div>
                        <button onclick="addConstraint('conflict')" class="mt-4 w-full border border-dashed border-gray-300 hover:border-red-500 text-gray-600 hover:text-red-600 py-2 rounded-lg transition duration-150 flex items-center justify-center">
                            <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/24/24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
                            Add Constraint
                        </button>
                    </div>
                </div>

            </div>

            <!-- Configuration and Results Column (1/3 width on large screens) -->
            <div class="space-y-8">
                
                <!-- Group Configuration -->
                <div class="card">
                    <h2 class="text-xl font-bold text-gray-800 mb-4">Group Configuration</h2>
                    <label for="num-groups" class="block text-sm font-medium text-gray-700 mb-2">Number of Groups</label>
                    <input type="number" id="num-groups" value="3" min="1" class="input-field mb-6">
                    
                    <button id="generate-button" onclick="generateGroups()" class="btn-primary w-full flex items-center justify-center">
                        <svg class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2l4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.002 12.002 0 0012 22a12.002 12.002 0 008.618-18.966z"></path></svg>
                        Generate Groups
                    </button>

                    <div id="message-box" class="mt-4 p-3 rounded-lg hidden text-sm font-medium"></div>
                </div>

                <!-- Group Results -->
                <div class="card" id="results-card" style="display: none;">
                    <h2 class="text-xl font-bold text-gray-800 mb-4">Generated Groups</h2>
                    
                    <!-- Export Buttons -->
                    <div class="export-actions flex gap-3 mb-4">
                        <button onclick="exportToCSV()" class="btn-export flex-1 flex items-center justify-center">
                            <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.885 2.5l-.837.837m0 0L3 18.5m10.5-5.5a4 4 0 11-8 0 4 4 0 018 0zM12 18V6m0 0l-3 3m3-3l3 3"></path></svg>
                            Export to CSV
                        </button>
                        <button id="export-pdf-button" onclick="exportToPDF()" class="btn-pdf flex-1 flex items-center justify-center">
                            <div id="pdf-export-content" class="flex items-center">
                                <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 13h6m-3-3v6m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.002 12.002 0 0012 22a12.002 12.002 0 008.618-18.966z"></path></svg>
                                Export to PDF
                            </div>
                        </button>
                    </div>

                    <div id="group-results" class="space-y-4">
                        <!-- Group cards will be injected here -->
                    </div>
                </div>

            </div>
        </div>
    </div>

    <script>
        // Global state
        let students = [];
        let mustBeTogether = []; // Array of arrays: [[id1, id2], [id3, id4]]
        let conflictConstraints = []; // Array of arrays: [[id1, id2], [id3, id4]]
        let lastGeneratedGroups = []; // Store the last successful generation for export

        const ACADEMIC_SCORES = { 'A': 5, 'B': 4, 'C': 3, 'D': 2, 'E': 1 };
        const BEHAVIOR_OPTIONS = [5, 4, 3, 2, 1];
        const ACADEMIC_OPTIONS = ['A', 'B', 'C', 'D', 'E'];
        const ROLE_OPTIONS = ['Other', 'Leader', 'Bully', 'Victim'];


        /**
         * Utility to generate a unique ID
         */
        const generateId = () => Math.random().toString(36).substring(2, 9);


        /**
         * RENDERING FUNCTIONS
         */

        /**
         * Renders the behavior dropdown for a student.
         */
        const renderBehaviorDropdown = (studentId, currentVal) => {
            let options = BEHAVIOR_OPTIONS.map(val => 
                `<option value="${val}" ${currentVal == val ? 'selected' : ''}>${val}</option>`
            ).join('');
            return `
                <select onchange="updateStudent('${studentId}', 'behavior', this.value)" class="dropdown">
                    ${options}
                </select>
            `;
        };

        /**
         * Renders the academic dropdown for a student.
         */
        const renderAcademicDropdown = (studentId, currentVal) => {
            let options = ACADEMIC_OPTIONS.map(val => 
                `<option value="${val}" ${currentVal === val ? 'selected' : ''}>${val}</option>`
            ).join('');
            return `
                <select onchange="updateStudent('${studentId}', 'academics', this.value)" class="dropdown">
                    ${options}
                </select>
            `;
        };
        
        /**
         * Renders the role dropdown for a student.
         */
        const renderRoleDropdown = (studentId, currentVal) => {
            let options = ROLE_OPTIONS.map(val => 
                `<option value="${val}" ${currentVal === val ? 'selected' : ''}>${val}</option>`
            ).join('');
            return `
                <select onchange="updateStudent('${studentId}', 'role', this.value)" class="dropdown">
                    ${options}
                </select>
            `;
        };

        /**
         * Renders the entire student roster list.
         */
        const renderStudentList = () => {
            const listContainer = document.getElementById('student-list');
            listContainer.innerHTML = '';
            
            if (students.length === 0) {
                document.getElementById('empty-roster-message').classList.remove('hidden');
            } else {
                document.getElementById('empty-roster-message').classList.add('hidden');
            }

            students.forEach(student => {
                const row = document.createElement('tr');
                row.className = 'hover:bg-gray-50';
                row.innerHTML = `
                    <td class="p-3 whitespace-nowrap">
                        <input type="text" value="${student.name}" onchange="updateStudent('${student.id}', 'name', this.value)" class="input-field max-w-40 text-sm font-medium text-gray-900"/>
                    </td>
                    <td class="p-3 whitespace-nowrap">
                        ${renderBehaviorDropdown(student.id, student.behavior)}
                    </td>
                    <td class="p-3 whitespace-nowrap">
                        ${renderAcademicDropdown(student.id, student.academics)}
                    </td>
                    <td class="p-3 whitespace-nowrap">
                        ${renderRoleDropdown(student.id, student.role)}
                    </td>
                    <td class="p-3 whitespace-nowrap text-sm font-medium">
                        <button onclick="removeStudent('${student.id}')" class="text-red-600 hover:text-red-900 p-1 rounded-full hover:bg-red-50 transition">
                            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                        </button>
                    </td>
                `;
                listContainer.appendChild(row);
            });
            renderConstraintSelectors();
        };

        /**
         * Renders a dropdown containing all current students.
         */
        const renderStudentSelector = (currentStudentId) => {
            const studentOptions = students.map(s => 
                `<option value="${s.id}" ${s.id === currentStudentId ? 'selected' : ''}>${s.name}</option>`
            ).join('');
            return `
                <select class="dropdown w-full" data-student-id="${currentStudentId || ''}">
                    <option value="">-- Select Student --</option>
                    ${studentOptions}
                </select>
            `;
        };

        /**
         * Renders all constraint sections (Must Be Together and Conflicts).
         */
        const renderConstraintSelectors = () => {
            const togetherList = document.getElementById('must-be-together-list');
            const conflictList = document.getElementById('conflict-constraints-list');
            
            togetherList.innerHTML = mustBeTogether.map((constraint, index) => 
                renderConstraintRow(constraint, index, 'together')
            ).join('');

            conflictList.innerHTML = conflictConstraints.map((constraint, index) => 
                renderConstraintRow(constraint, index, 'conflict')
            ).join('');
        };

        /**
         * Renders a single constraint row.
         */
        const renderConstraintRow = (constraint, index, type) => {
            const baseId = `${type}-${index}`;
            const relationText = type === 'together' ? 'and' : 'must not be with'; 
            
            return `
                <div class="flex items-center space-x-2 p-3 bg-gray-50 rounded-lg border" id="${baseId}">
                    <div class="flex-grow space-y-1">
                        ${renderStudentSelector(constraint[0], baseId + '-0')}
                        <span class="text-xs text-gray-500 block text-center">${relationText}</span>
                        ${renderStudentSelector(constraint[1], baseId + '-1')}
                    </div>
                    <button onclick="removeConstraint(${index}, '${type}')" class="text-red-500 hover:text-red-700 p-1 transition" title="Remove constraint">
                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                    </button>
                </div>
            `;
        };


        /**
         * STATE MANAGEMENT FUNCTIONS
         */

        /**
         * Adds a new student row to the state.
         */
        const addStudentRow = () => {
            const newStudent = {
                id: generateId(),
                name: `Student ${students.length + 1}`,
                behavior: 3,
                academics: 'C',
                role: 'Other'
            };
            students.push(newStudent);
            renderStudentList();
        };

        /**
         * Updates a student property in the state.
         */
        const updateStudent = (id, key, value) => {
            const student = students.find(s => s.id === id);
            if (student) {
                // Parse value if it's a number
                student[key] = key === 'behavior' ? parseInt(value) : value;
                // Re-render only to update constraint selectors if name changed
                if (key === 'name') {
                    renderStudentList();
                }
            }
        };

        /**
         * Removes a student from the state and all constraints.
         */
        const removeStudent = (id) => {
            students = students.filter(s => s.id !== id);
            
            // Remove from constraints
            mustBeTogether = mustBeTogether.filter(c => !c.includes(id));
            conflictConstraints = conflictConstraints.filter(c => !c.includes(id));

            renderStudentList();
        };
        
        /**
         * Adds a new two-student constraint to the state.
         */
        const addConstraint = (type) => {
            const constraintList = type === 'together' ? mustBeTogether : conflictConstraints;
            // Add a constraint with two empty placeholders
            constraintList.push(["", ""]); 
            renderConstraintSelectors();

            // Need to attach listeners after rendering to update state on change
            const listElement = document.getElementById(type === 'together' ? 'must-be-together-list' : 'conflict-constraints-list');
            const newConstraintDiv = listElement.lastElementChild;
            if (newConstraintDiv) {
                newConstraintDiv.querySelectorAll('select').forEach((select, index) => {
                    select.addEventListener('change', (e) => {
                        const constraintIndex = Array.from(listElement.children).indexOf(newConstraintDiv);
                        const listToUpdate = type === 'together' ? mustBeTogether : conflictConstraints;
                        listToUpdate[constraintIndex][index] = e.target.value;
                    });
                });
            }
        };

        /**
         * Removes a constraint from the state.
         */
        const removeConstraint = (index, type) => {
            if (type === 'together') {
                mustBeTogether.splice(index, 1);
            } else {
                conflictConstraints.splice(index, 1);
            }
            renderConstraintSelectors();
        };

        /**
         * HELPER FUNCTIONS FOR GROUPING
         */

        /**
         * Converts academic grade to numerical score (A=5 to E=1).
         */
        const getAcademicScore = (grade) => ACADEMIC_SCORES[grade] || 0;

        /**
         * Checks if adding a student to a group violates any conflict constraints 
         * (explicit conflicts OR implicit Bully/Victim conflict).
         */
        const hasConflict = (student, group, allConstraints) => {
            // 1. Check Explicit Constraints
            const studentId = student.id;
            const studentConflicts = allConstraints.filter(c => c.includes(studentId));

            for (const conflictPair of studentConflicts) {
                const otherId = conflictPair.find(id => id !== studentId);
                if (group.some(s => s.id === otherId)) {
                    return true;
                }
            }

            // 2. Check Implicit Bully/Victim Constraint (Universal Hard Constraint)
            const newStudentRole = student.role;
            const isNewStudentBully = newStudentRole === 'Bully';
            const isNewStudentVictim = newStudentRole === 'Victim';

            for (const groupMember of group) {
                const groupMemberRole = groupMember.role;

                if ((isNewStudentBully && groupMemberRole === 'Victim') ||
                    (isNewStudentVictim && groupMemberRole === 'Bully')) {
                    return true;
                }
            }

            return false;
        };
        
        /**
         * Calculates the total score (Behavior + Academic) for a group.
         */
        const calculateGroupScore = (group) => {
            let totalBehavior = 0;
            let totalAcademic = 0;
            let leaderCount = 0;

            for (const student of group) {
                totalBehavior += student.behavior;
                totalAcademic += student.academicScore;
                if (student.role === 'Leader') {
                    leaderCount++;
                }
            }
            return { totalBehavior, totalAcademic, leaderCount, totalScore: totalBehavior + totalAcademic, size: group.length };
        };

        /**
         * Main grouping logic using a greedy, heuristic approach.
         */
        const generateGroups = () => {
            const numGroups = parseInt(document.getElementById('num-groups').value);
            const messageBox = document.getElementById('message-box');
            messageBox.style.display = 'none';

            // 1. Validation and Setup
            if (students.length === 0) {
                showMessage("error", "Please add at least one student to the roster.");
                return;
            }
            if (numGroups <= 0 || numGroups > students.length) {
                showMessage("error", "Number of groups must be between 1 and the total number of students.");
                return;
            }
            
            // Clean up and preprocess students
            const allStudents = students.map(s => ({
                ...s,
                behavior: parseInt(s.behavior),
                academicScore: getAcademicScore(s.academics)
            }));
            
            // Check for valid constraints (only students with IDs are considered)
            const validMustBeTogether = mustBeTogether.filter(c => allStudents.some(s => s.id === c[0]) && allStudents.some(s => s.id === c[1]));
            const validConflictConstraints = conflictConstraints.filter(c => allStudents.some(s => s.id === c[0]) && allStudents.some(s => s.id === c[1]));

            // 2. Initialize Groups
            let groups = Array.from({ length: numGroups }, () => []);
            let assignedStudentIds = new Set();
            
            // 3. Phase 1: Hard Constraints (Must Be Together)
            for (const constraint of validMustBeTogether) {
                const [id1, id2] = constraint;
                const s1 = allStudents.find(s => s.id === id1);
                const s2 = allStudents.find(s => s.id === id2);

                if (assignedStudentIds.has(id1) || assignedStudentIds.has(id2)) {
                    showMessage("warning", "A 'Must Be Together' constraint involves a student already assigned by a previous constraint. This constraint might be partially ignored.");
                    continue; 
                }

                // CHECK: Bully/Victim conflict between the pair themselves
                if (hasConflict(s1, [s2], validConflictConstraints)) {
                    showMessage("error", `Cannot satisfy 'Must Be Together' constraint: ${s1.name} and ${s2.name} have an inherent or explicit conflict (e.g., Bully/Victim).`);
                    return;
                }

                let bestGroupIndex = -1;
                let minSize = Infinity;

                for (let i = 0; i < numGroups; i++) {
                    if (groups[i].length < minSize) {
                        // Check if adding both students violates any conflict constraints with current group members
                        if (!hasConflict(s1, groups[i], validConflictConstraints) && !hasConflict(s2, groups[i], validConflictConstraints)) {
                             minSize = groups[i].length;
                             bestGroupIndex = i;
                        }
                    }
                }

                if (bestGroupIndex !== -1) {
                    groups[bestGroupIndex].push(s1, s2);
                    assignedStudentIds.add(id1);
                    assignedStudentIds.add(id2);
                } else {
                    showMessage("error", `Cannot satisfy a 'Must Be Together' constraint (${s1.name} and ${s2.name}) because it creates an unavoidable conflict with an existing group member.`);
                    return; 
                }
            }
            
            // 4. Phase 2: Greedy Assignment of Remaining Students
            let unassignedStudents = allStudents.filter(s => !assignedStudentIds.has(s.id));
            
            // Sort students: Leaders/Bullies/Victims first, then by overall score.
            unassignedStudents.sort((a, b) => {
                // Roles that need spreading or careful placement go first
                const roleOrder = (role) => {
                    if (role === 'Leader') return 3;
                    if (role === 'Bully' || role === 'Victim') return 2;
                    return 1;
                };
                if (roleOrder(a.role) !== roleOrder(b.role)) return roleOrder(b.role) - roleOrder(b.role);
                
                // Then by overall score (high score first to fill groups early)
                return (b.behavior + b.academicScore) - (a.behavior + a.academicScore);
            });

            for (const student of unassignedStudents) {
                let bestGroupIndex = -1;
                let minGroupSize = Infinity;
                let minScoreVariance = Infinity;
                
                // Calculate the target average score (Behavior + Academic) per group
                const totalScoreAll = allStudents.reduce((sum, s) => sum + s.behavior + s.academicScore, 0);
                const targetAvgScore = totalScoreAll / numGroups;
                
                // Find the best group to place the student
                for (let i = 0; i < numGroups; i++) {
                    const currentGroup = groups[i];
                    
                    // 4a. Check Hard Conflict Constraint (Explicit AND Bully/Victim)
                    if (hasConflict(student, currentGroup, validConflictConstraints)) {
                        continue; // Skip this group, conflict violation
                    }

                    // Tentative group with new student
                    const tentativeGroup = [...currentGroup, student];
                    const tentativeScore = calculateGroupScore(tentativeGroup);
                    
                    // 4b. Calculate Metrics for Comparison
                    const groupSize = tentativeGroup.length;
                    // Measures how far the group's total score is from a perfectly balanced score
                    const scoreDifference = Math.abs(tentativeScore.totalScore - (targetAvgScore * groupSize)); 
                    
                    // Prioritization:
                    // 1. Group Size (absolute priority)
                    if (groupSize < minGroupSize) {
                        bestGroupIndex = i;
                        minGroupSize = groupSize;
                        minScoreVariance = scoreDifference;
                    } 
                    // 2. Score Balance (secondary priority if sizes are equal)
                    else if (groupSize === minGroupSize && scoreDifference < minScoreVariance) {
                        bestGroupIndex = i;
                        minScoreVariance = scoreDifference;
                    } 
                    // 3. Leader Spread (tie-breaker for size/score balance - send leader to group with fewest leaders)
                    else if (groupSize === minGroupSize && scoreDifference === minScoreVariance && student.role === 'Leader') {
                        const currentGroupScore = calculateGroupScore(groups[bestGroupIndex]);
                        const targetGroupScore = calculateGroupScore(currentGroup);
                        if (targetGroupScore.leaderCount < currentGroupScore.leaderCount) {
                            bestGroupIndex = i;
                        }
                    }
                }

                if (bestGroupIndex !== -1) {
                    groups[bestGroupIndex].push(student);
                    assignedStudentIds.add(student.id);
                } else {
                    // This should only happen if all groups have a conflict, which is a rare but possible scenario
                    // if constraints are too restrictive (e.g., student A must be separate from everyone).
                    showMessage("error", `Could not place student ${student.name} into any group without violating a conflict constraint (explicit or Bully/Victim). Grouping stopped.`);
                    return;
                }
            }

            // 5. Save and Render Results
            lastGeneratedGroups = groups;
            renderGroupResults(groups);
            showMessage("success", `Successfully generated ${numGroups} balanced groups.`);
        };


        /**
         * RENDERING RESULTS
         */
        const renderGroupResults = (groups) => {
            const resultsContainer = document.getElementById('group-results');
            resultsContainer.innerHTML = '';
            document.getElementById('results-card').style.display = 'block';

            groups.forEach((group, index) => {
                const { totalBehavior, totalAcademic, leaderCount } = calculateGroupScore(group);
                
                const groupElement = document.createElement('div');
                groupElement.className = 'bg-blue-50 p-4 rounded-lg border border-blue-200';
                groupElement.innerHTML = `
                    <h3 class="text-lg font-semibold text-blue-800 mb-2">Group ${index + 1} <span class="text-sm font-normal text-blue-600">(${group.length} students)</span></h3>
                    
                    <div class="flex flex-wrap gap-4 text-sm mb-3">
                        <div class="flex items-center text-gray-700">
                            <svg class="w-4 h-4 mr-1 text-green-500" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg"><path d="M5 3a2 2 0 00-2 2v2a2 2 0 002 2h2a2 2 0 002-2V5a2 2 0 00-2-2H5zm0 10a2 2 0 00-2 2v2a2 2 0 002 2h2a2 2 0 002-2v-2a2 2 0 00-2-2H5zM13 3a2 2 0 00-2 2v2a2 2 0 002 2h2a2 2 0 002-2V5a2 2 0 00-2-2h-2zm0 10a2 2 0 00-2 2v2a2 2 0 002 2h2a2 2 0 002-2v-2a2 2 0 00-2-2h-2z"></path></svg>
                            Total Behavior Score: <span class="font-medium ml-1">${totalBehavior}</span>
                        </div>
                        <div class="flex items-center text-gray-700">
                            <svg class="w-4 h-4 mr-1 text-yellow-500" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" d="M10 2a1 1 0 011 1v1.5a1 1 0 01-2 0V3a1 1 0 011-1zm6.364 2.636a1 1 0 01-.707 1.707L14.95 6.05a1 1 0 01-1.414-1.414l.707-.707a1 1 0 011.414 0zM17 10a1 1 0 01-1 1h-1.5a1 1 0 010-2H16a1 1 0 011 1zm-2.636 5.364a1 1 0 01-1.414.071l-.707-.707a1 1 0 01.071-1.414l.707.707a1 1 0 011.414 1.414zM10 16a1 1 0 01-1 1H7a1 1 0 01-1-1v-1.5a1 1 0 012 0V16a1 1 0 011 1zM3.636 4.636a1 1 0 011.707-.707L6.05 4.95a1 1 0 01-1.414 1.414l-.707-.707a1 1 0 010-1.414zM3 10a1 1 0 011-1h1.5a1 1 0 010 2H4a1 1 0 01-1-1zm2.636 5.364a1 1 0 01-1.414 0l-.707-.707a1 1 0 011.414-1.414l.707.707a1 1 0 010 1.414z" clip-rule="evenodd"></path></svg>
                            Total Academic Score: <span class="font-medium ml-1">${totalAcademic}</span>
                        </div>
                         <div class="flex items-center text-gray-700">
                            <svg class="w-4 h-4 mr-1 text-purple-500" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" d="M10 9a3 3 0 100-6 3 3 0 000 6zm-7 9a7 7 0 1114 0H3z" clip-rule="evenodd"></path></svg>
                            Leaders: <span class="font-medium ml-1">${leaderCount}</span>
                        </div>
                    </div>

                    <ul class="list-disc pl-5 space-y-1 text-gray-800">
                        ${group.map(s => {
                            let roleText = '';
                            // Only add role if it's Leader, Bully, or Victim
                            if (['Leader', 'Bully', 'Victim'].includes(s.role)) {
                                roleText = `<span class="text-xs ml-1 text-gray-600">(${s.role})</span>`;
                            }
                            
                            // Display format: Name BScoreAGrade (Role)
                            return `
                                <li class="text-sm">
                                    ${s.name} <span class="font-semibold text-blue-700">${s.behavior}${s.academics}</span>${roleText}
                                </li>
                            `;
                        }).join('')}
                    </ul>
                `;
                resultsContainer.appendChild(groupElement);
            });
        };

        /**
         * Renders a message box for success or error.
         */
        const showMessage = (type, message) => {
            const box = document.getElementById('message-box');
            box.textContent = message;
            box.classList.remove('hidden', 'bg-red-100', 'border-red-500', 'text-red-800', 'bg-green-100', 'border-green-500', 'text-green-800', 'bg-yellow-100', 'border-yellow-500', 'text-yellow-800');

            if (type === 'error') {
                box.classList.add('bg-red-100', 'border-red-500', 'text-red-800', 'border');
            } else if (type === 'success') {
                box.classList.add('bg-green-100', 'border-green-500', 'text-green-800', 'border');
            } else if (type === 'warning') {
                box.classList.add('bg-yellow-100', 'border-yellow-500', 'text-yellow-800', 'border');
            }
            box.style.display = 'block';
        };

        /**
         * EXPORT FUNCTIONS
         */

        /**
         * Exports the generated groups to a CSV file.
         */
        const exportToCSV = () => {
            if (lastGeneratedGroups.length === 0) {
                showMessage('warning', 'Please generate groups first before attempting to export.');
                return;
            }

            let csvContent = "Group,Name,Behavior Score,Academic Grade,Role\n";

            lastGeneratedGroups.forEach((group, groupIndex) => {
                const groupNumber = `Group ${groupIndex + 1}`;
                group.forEach(student => {
                    // Escape commas in student names just in case
                    const name = student.name.replace(/"/g, '""');
                    csvContent += `"${groupNumber}","${name}",${student.behavior},${student.academics},"${student.role}"\n`;
                });
            });

            const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
            const url = URL.createObjectURL(blob);
            const link = document.createElement('a');
            link.setAttribute('href', url);
            link.setAttribute('download', 'student_groups.csv');
            link.style.visibility = 'hidden';
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            showMessage('success', 'Groups successfully exported to CSV.');
        };
        
        /**
         * Generates and downloads a PDF file from the results card using html2canvas and jsPDF.
         */
        const exportToPDF = async () => {
            if (lastGeneratedGroups.length === 0) {
                showMessage('warning', 'Please generate groups first before attempting to export.');
                return;
            }

            const resultsCard = document.getElementById('results-card');
            const exportActions = resultsCard.querySelector('.export-actions');
            const pdfButtonContent = document.getElementById('pdf-export-content');
            
            // 1. Show Loading State
            pdfButtonContent.innerHTML = '<div class="spinner"></div><span class="ml-2">Generating PDF...</span>';
            exportActions.style.display = 'none'; // Hide buttons during capture

            try {
                // FIX: Correctly access the jsPDF class constructor from the window object's jspdf UMD property
                const JsPDF = window.jspdf.jsPDF; 

                // 2. Capture the HTML element as a canvas image
                const canvas = await html2canvas(resultsCard, {
                    scale: 2, // Increase scale for higher quality image in PDF
                    logging: false,
                    allowTaint: true,
                    useCORS: true,
                    // Temporarily set background to white for print clarity if it was transparent
                    backgroundColor: '#ffffff' 
                });

                const imgData = canvas.toDataURL('image/png');
                const imgWidth = 200; // Standard A4 width in mm (approx)
                const pageHeight = 295; // Standard A4 height in mm (approx)
                const imgHeight = canvas.height * imgWidth / canvas.width;
                let heightLeft = imgHeight;
                let position = 0;

                // 3. Initialize jsPDF (A4 size, units in mm) using the corrected constructor
                const pdf = new JsPDF('p', 'mm', 'a4');

                // 4. Add Image to PDF, handling multiple pages if content is long
                // Add first page
                pdf.addImage(imgData, 'PNG', 5, position + 5, imgWidth - 10, imgHeight);
                heightLeft -= pageHeight;

                // Add subsequent pages if the content height exceeds one page
                while (heightLeft >= 0) {
                    position = heightLeft - imgHeight;
                    pdf.addPage();
                    pdf.addImage(imgData, 'PNG', 5, position + 5, imgWidth - 10, imgHeight);
                    heightLeft -= pageHeight;
                }

                // 5. Save the PDF
                pdf.save('student_groups.pdf');

                showMessage('success', 'Groups successfully exported to PDF.');

            } catch (error) {
                console.error("PDF generation failed:", error);
                showMessage('error', 'Failed to generate PDF. See console for details.');
            } finally {
                // 6. Restore Button State
                pdfButtonContent.innerHTML = `
                    <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 13h6m-3-3v6m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.002 12.002 0 0012 22a12.002 12.002 0 008.618-18.966z"></path></svg>
                    Export to PDF
                `;
                exportActions.style.display = 'flex';
            }
        };


        /**
         * INITIALIZATION AND DUMMY DATA
         */

        const initializeApp = () => {
            // Dummy data for the student grouping app
            students = [
                { id: generateId(), name: 'Alice', behavior: 5, academics: 'A', role: 'Leader' },
                { id: generateId(), name: 'Bob', behavior: 1, academics: 'E', role: 'Victim' },
                { id: generateId(), name: 'Charlie', behavior: 3, academics: 'C', role: 'Bully' },
                { id: generateId(), name: 'David', behavior: 4, academics: 'B', role: 'Other' },
                { id: generateId(), name: 'Eve', behavior: 2, academics: 'D', role: 'Other' },
                { id: generateId(), name: 'Frank', behavior: 5, academics: 'A', role: 'Leader' },
                { id: generateId(), name: 'Grace', behavior: 1, academics: 'E', role: 'Other' },
                { id: generateId(), name: 'Henry', behavior: 3, academics: 'C', role: 'Bully' },
                { id: generateId(), name: 'Ivy', behavior: 2, academics: 'D', role: 'Victim' },
            ].map(s => ({ ...s, academicScore: getAcademicScore(s.academics) }));

            renderStudentList();
        };

        window.onload = initializeApp;
        
    </script>
</body>
</html>
