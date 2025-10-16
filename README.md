@php
    // Doctor-specific statistics
    $doctorId = auth()->user()->id;
    $totalPatients = \App\Patient::count();
    
    // Patient Card Type Statistics
    $ancPatients = \App\Patient::where('card_type', 'ANC')->count();
    $emergencyPatients = \App\Patient::where('card_type', 'Emergency')->count();
    $personalPatients = \App\Patient::where('card_type', 'Personal')->count();
    $familyPatients = \App\Patient::where('card_type', 'Family')->count();
    $ashiaPatients = \App\Patient::where('card_type', 'ASHIA')->count();
    
    // Patient Type Statistics
    $inPatients = \App\Patient::where('patient_type', 'in-patient')->count();
    $outPatients = \App\Patient::where('patient_type', 'out-patient')->count();
    $dayPatients = \App\Patient::where('patient_type', 'day-patient')->count();
    $totalAppointments = \App\Appointment::where('status', '!=', 'vitals_only')->count();
    $activeAppointments = \App\Appointment::where('status', 'in_consultation')->count();
    $completedAppointments = \App\Appointment::where('status', 'completed')->count();
    $pendingAppointments = \App\Appointment::where('status', 'pending')->count();
    
    $totalPrescriptions = \App\Prescription::count();
    $activePrescriptions = \App\Prescription::where('status', 'active')
        ->whereHas('appointment', function($q) {
            $q->where('in_treatment', true);
        })->count();
    $completedPrescriptions = \App\Prescription::where('status', 'completed')->count();
    $discontinuedPrescriptions = \App\Prescription::where('status', 'discontinued')->count();
    
    // Appointments forwarded to doctor that need consultation
    $appointmentsNeedingConsultation = \App\Appointment::where('status', 'in_consultation')
        ->where('status', '!=', 'vitals_only')
        ->with(['patient', 'doctor'])
        ->orderBy('created_at', 'desc')
        ->limit(5)
        ->get();
    
    // Appointments without prescriptions (need prescription)
    $appointmentsWithoutPrescriptions = \App\Appointment::where('status', '!=', 'vitals_only')
        ->whereDoesntHave('prescriptions')
        ->with(['patient', 'doctor'])
        ->orderBy('created_at', 'desc')
        ->limit(5)
        ->get();
@endphp

<div class="container-fluid">
<div class="container-fluid px-3">
    <h3 class="fw-bold text-info mb-4">
        <i class="bi bi-heart-pulse me-2"></i> Doctor Dashboard
    </h3>

    <div class="row g-4">
        <!-- Patient Management Card -->
        <div class="col-md-4">
            <div class="card shadow-sm border-0 h-100 dashboard-card">
                <div class="card-body d-flex flex-column">
                    <div class="d-flex align-items-center mb-3">
                        <div class="bg-info text-white rounded-circle d-flex align-items-center justify-content-center p-2 me-3" style="width: 64px; height: 64px;">
                            <i class="bi bi-person-badge fs-4"></i>
                        </div>
                        <div>
                            <h5 class="fw-semibold mb-0">Patient Records</h5>
                            <small class="text-muted">Total: {{ $totalPatients }}</small>
                        </div>
                    </div>

                    <!-- Card Type Statistics -->
                    <div class="mb-3">
                        <div class="row g-2">
                            <div class="col">
                                <div class="bg-primary bg-opacity-10 rounded p-2 text-center">
                                    <small class="text-primary fw-semibold d-block">ANC</small>
                                    <span class="text-primary fw-bold">{{ $ancPatients }}</span>
                                </div>
                            </div>
                            <div class="col">
                                <div class="bg-danger bg-opacity-10 rounded p-2 text-center">
                                    <small class="text-danger fw-semibold d-block">Emergency</small>
                                    <span class="text-danger fw-bold">{{ $emergencyPatients }}</span>
                                </div>
                            </div>
                            <div class="col">
                                <div class="bg-success bg-opacity-10 rounded p-2 text-center">
                                    <small class="text-success fw-semibold d-block">Personal</small>
                                    <span class="text-success fw-bold">{{ $personalPatients }}</span>
                                </div>
                            </div>
                            <div class="col">
                                <div class="bg-info bg-opacity-10 rounded p-2 text-center">
                                    <small class="text-info fw-semibold d-block">Family</small>
                                    <span class="text-info fw-bold">{{ $familyPatients }}</span>
                                </div>
                            </div>
                            <div class="col">
                                <div class="bg-warning bg-opacity-10 rounded p-2 text-center">
                                    <small class="text-warning fw-semibold d-block">ASHIA</small>
                                    <span class="text-warning fw-bold">{{ $ashiaPatients }}</span>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div class="mt-auto">
                        <button id="show-patient-list" class="btn btn-info btn-sm d-flex align-items-center">
                            <i class="bi bi-people me-1"></i> View Patients
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Appointments Card -->
        <div class="col-md-4">
            <div class="card shadow-sm border-0 h-100 dashboard-card">
                <div class="card-body d-flex flex-column">
                    <div class="d-flex align-items-center mb-3">
                        <div class="bg-success text-white rounded-circle d-flex align-items-center justify-content-center p-2 me-3" style="width: 64px; height: 64px;">
                            <i class="bi bi-calendar-check fs-4"></i>
                        </div>
                        <div>
                            <h5 class="fw-semibold mb-0">Appointments</h5>
                            <small class="text-muted">Total: {{ $totalAppointments }}</small>
                        </div>
                    </div>

                    <div class="row g-2 mb-3">
                        <div class="col-4">
                            <div class="bg-light rounded p-2 text-center">
                                <small class="text-muted d-block">Active</small>
                                <strong class="text-warning">{{ $activeAppointments }}</strong>
                            </div>
                        </div>
                        <div class="col-4">
                            <div class="bg-light rounded p-2 text-center">
                                <small class="text-muted d-block">Completed</small>
                                <strong class="text-success">{{ $completedAppointments }}</strong>
                            </div>
                        </div>
                        <div class="col-4">
                            <div class="bg-light rounded p-2 text-center">
                                <small class="text-muted d-block">Pending</small>
                                <strong class="text-info">{{ $pendingAppointments }}</strong>
                            </div>
                        </div>
                    </div>

                    <div class="mt-auto">
                        <button id="show-appointment-list" class="btn btn-success btn-sm d-flex align-items-center">
                            <i class="bi bi-calendar-check me-1"></i> View Appointments
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Prescriptions Card -->
        <div class="col-md-4">
            <div class="card shadow-sm border-0 h-100 dashboard-card">
                <div class="card-body d-flex flex-column">
                    <div class="d-flex align-items-center mb-3">
                        <div class="bg-primary text-white rounded-circle d-flex align-items-center justify-content-center p-2 me-3" style="width: 64px; height: 64px;">
                            <i class="bi bi-capsule fs-4"></i>
                        </div>
                        <div>
                            <h5 class="fw-semibold mb-0">Prescriptions</h5>
                            <small class="text-muted">Total: {{ $totalPrescriptions }}</small>
                        </div>
                    </div>

                    <div class="row g-2 mb-3">
                        <div class="col-4">
                            <div class="bg-light rounded p-2 text-center">
                                <small class="text-muted d-block">Active</small>
                                <strong class="text-warning">{{ $activePrescriptions }}</strong>
                            </div>
                        </div>
                        <div class="col-4">
                            <div class="bg-light rounded p-2 text-center">
                                <small class="text-muted d-block">Completed</small>
                                <strong class="text-success">{{ $completedPrescriptions }}</strong>
                            </div>
                        </div>
                        <div class="col-4">
                            <div class="bg-light rounded p-2 text-center">
                                <small class="text-muted d-block">Total</small>
                                <strong class="text-primary">{{ $totalPrescriptions }}</strong>
                            </div>
                        </div>
                    </div>

                    <div class="mt-auto">
                        <button id="show-prescription-list" class="btn btn-primary btn-sm d-flex align-items-center">
                            <i class="bi bi-capsule me-1"></i> View Prescriptions
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Appointments Needing Consultation Card -->
        <div class="col-md-6">
            <div class="card shadow-sm border-0 h-100 dashboard-card">
                <div class="card-body d-flex flex-column">
                    <div class="d-flex align-items-center mb-3">
                        <div class="bg-success text-white rounded-circle d-flex align-items-center justify-content-center p-2 me-3" style="width: 64px; height: 64px;">
                            <i class="bi bi-person-check fs-4"></i>
                        </div>
                        <div>
                            <h5 class="fw-semibold mb-0">Consultation Needed</h5>
                            <small class="text-muted">Appointments forwarded to doctor</small>
                        </div>
                    </div>

                    <div class="flex-grow-1">
                        @if($appointmentsNeedingConsultation->count() > 0)
                            <div class="list-group list-group-flush">
                                @foreach($appointmentsNeedingConsultation as $appointment)
                                    <div class="list-group-item px-0 py-2 border-0 appointment-item" 
                                         style="cursor: pointer;" 
                                         onclick="loadAjaxView('/doctor/appointments/{{ $appointment->id }}')">
                                        <div class="d-flex justify-content-between align-items-center">
                                            <div>
                                                <h6 class="mb-1 fw-semibold">
                                                    @if($appointment->patient)
                                                        {{ $appointment->patient->surname }} {{ $appointment->patient->other_names }}
                                                    @else
                                                        <span class="text-danger">Patient Not Found (ID: {{ $appointment->patient_id }})</span>
                                                    @endif
                                                </h6>
                                                <small class="text-muted">
                                                    <i class="bi bi-calendar3 me-1"></i>
                                                    {{ $appointment->date->format('M d, Y') }}
                                                    <span class="mx-2">•</span>
                                                    <i class="bi bi-clock me-1"></i>
                                                    {{ $appointment->date->format('h:i A') }}
                                                </small>
                                            </div>
                                            <div class="text-end">
                                                <span class="badge 
                                                    @if($appointment->status === 'completed') bg-success
                                                    @elseif($appointment->status === 'in_consultation') bg-info
                                                    @else bg-secondary
                                                    @endif">
                                                    {{ ucfirst($appointment->status) }}
                                                </span>
                                                @if($appointment->diagnosis)
                                                    <div class="mt-1">
                                                        <small class="text-info">
                                                            <i class="bi bi-clipboard-check me-1"></i>
                                                            Diagnosed
                                                        </small>
                                                    </div>
                                                @endif
                                            </div>
                                        </div>
                                    </div>
                                @endforeach
                            </div>
                        @else
                            <div class="text-center py-4">
                                <i class="bi bi-check-circle text-muted" style="font-size: 2rem;"></i>
                                <p class="text-muted mt-2 mb-0">No appointments need consultation</p>
                            </div>
                        @endif
                    </div>

                    <div class="mt-auto">
                        <button id="show-appointment-list" class="btn btn-success btn-sm d-flex align-items-center">
                            <i class="bi bi-person-check me-1"></i> View Consultations
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Prescriptions Needed Card -->
        <div class="col-md-6">
            <div class="card shadow-sm border-0 h-100 dashboard-card">
                <div class="card-body d-flex flex-column">
                    <div class="d-flex align-items-center mb-3">
                        <div class="bg-warning text-white rounded-circle d-flex align-items-center justify-content-center p-2 me-3" style="width: 64px; height: 64px;">
                            <i class="bi bi-capsule fs-4"></i>
                        </div>
                        <div>
                            <h5 class="fw-semibold mb-0">Prescriptions Needed</h5>
                            <small class="text-muted">Appointments without prescriptions</small>
                        </div>
                    </div>

                    <div class="flex-grow-1">
                        @if($appointmentsWithoutPrescriptions->count() > 0)
                            <div class="list-group list-group-flush">
                                @foreach($appointmentsWithoutPrescriptions as $appointment)
                                    <div class="list-group-item px-0 py-2 border-0 appointment-item" 
                                         style="cursor: pointer;" 
                                         onclick="loadAjaxView('/doctor/appointments/{{ $appointment->id }}')">
                                        <div class="d-flex justify-content-between align-items-center">
                                            <div>
                                                <h6 class="mb-1 fw-semibold">
                                                    @if($appointment->patient)
                                                        {{ $appointment->patient->surname }} {{ $appointment->patient->other_names }}
                                                    @else
                                                        <span class="text-danger">Patient Not Found (ID: {{ $appointment->patient_id }})</span>
                                                    @endif
                                                </h6>
                                                <small class="text-muted">
                                                    <i class="bi bi-calendar3 me-1"></i>
                                                    {{ $appointment->date->format('M d, Y') }}
                                                    <span class="mx-2">•</span>
                                                    <i class="bi bi-clock me-1"></i>
                                                    {{ $appointment->date->format('h:i A') }}
                                                </small>
                                                @if($appointment->diagnosis)
                                                    <div class="mt-1">
                                                        <small class="text-info">
                                                            <i class="bi bi-clipboard-check me-1"></i>
                                                            <div class="d-inline" style="max-height: 2em; overflow: hidden; line-height: 1em;">
                                                                {!! Str::limit($appointment->diagnosis, 100) !!}
                                                            </div>
                                                        </small>
                                                    </div>
                                                @endif
                                            </div>
                                            <div class="text-end">
                                                <span class="badge 
                                                    @if($appointment->status === 'completed') bg-success
                                                    @elseif($appointment->status === 'in_consultation') bg-info
                                                    @else bg-secondary
                                                    @endif">
                                                    {{ ucfirst($appointment->status) }}
                                                </span>
                                                <div class="mt-1">
                                                    <small class="text-warning">
                                                        <i class="bi bi-exclamation-triangle me-1"></i>
                                                        Needs Prescription
                                                    </small>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                @endforeach
                            </div>
                        @else
                            <div class="text-center py-4">
                                <i class="bi bi-check-circle text-muted" style="font-size: 2rem;"></i>
                                <p class="text-muted mt-2 mb-0">All appointments have prescriptions</p>
                            </div>
                        @endif
                    </div>

                    <div class="mt-auto">
                        <button id="show-prescription-list" class="btn btn-warning btn-sm d-flex align-items-center">
                            <i class="bi bi-capsule me-1"></i> Manage Prescriptions
                        </button>
                    </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div> 
